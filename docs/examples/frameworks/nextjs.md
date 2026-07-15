# Next.js Integration

This guide covers server-only App Router integration. The five supported example flows and their request semantics live in [Core Workflows](../curl/all-endpoints.md).

## Environment

```env
# .env.local — never prefix this with NEXT_PUBLIC_
PUBLORA_API_KEY=sk_YOUR_API_KEY
PUBLORA_BASE_URL=https://api.publora.com/api/v1
PUBLORA_WEBHOOK_SECRET=your_webhook_secret
```

## Server-side client

```ts
// lib/publora.ts
const baseUrl = process.env.PUBLORA_BASE_URL ?? 'https://api.publora.com/api/v1';

export async function publora<T>(
  path: string,
  init: RequestInit = {},
  idempotencyKey?: string,
): Promise<T> {
  const response = await fetch(`${baseUrl}/${path}`, {
    ...init,
    headers: {
      'content-type': 'application/json',
      'x-publora-key': process.env.PUBLORA_API_KEY!,
      ...(idempotencyKey ? { 'Idempotency-Key': idempotencyKey } : {}),
      ...init.headers,
    },
    cache: 'no-store',
  });
  const body = await response.json();
  if (!response.ok) throw new Error(body.error ?? `Publora returned ${response.status}`);
  return body as T;
}
```

## App Router endpoint

Keep the API key on the server and validate your own user's input before forwarding it.

```ts
// app/api/social-posts/route.ts
import { NextResponse } from 'next/server';
import { randomUUID } from 'node:crypto';
import { publora } from '@/lib/publora';

export async function POST(request: Request) {
  const { content, platforms, publish } = await request.json();
  const scheduledTime = publish
    ? new Date(Date.now() + 5 * 60_000).toISOString()
    : undefined;

  const result = await publora<{ success: boolean; postGroupId: string }>(
    'create-post',
    {
      method: 'POST',
      body: JSON.stringify({ content, platforms, scheduledTime }),
    },
    randomUUID(),
  );

  return NextResponse.json(result);
}
```

When `publish` is false, this intentionally creates a draft. A UI should label those choices “Create Draft” and “Schedule Post”; never call an empty-time submission scheduled.

## Raw-body webhook route

```ts
// app/api/publora-webhook/route.ts
import crypto from 'node:crypto';

export async function POST(request: Request) {
  const raw = Buffer.from(await request.arrayBuffer());
  const received = request.headers.get('x-publora-signature') ?? '';
  const expected = crypto
    .createHmac('sha256', process.env.PUBLORA_WEBHOOK_SECRET!)
    .update(raw)
    .digest('hex');

  const valid = received.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(received), Buffer.from(expected));
  if (!valid) return new Response(null, { status: 401 });

  const envelope = JSON.parse(raw.toString('utf8'));
  // Enqueue application work here.
  return new Response(null, { status: 204 });
}
```

Do not call `request.json()` before capturing the raw bytes. See [Webhooks](../../endpoints/webhooks.md).

## Workflow links

- [Draft](../curl/all-endpoints.md#1-create-a-draft)
- [One-shot schedule](../curl/all-endpoints.md#2-one-shot-schedule)
- [Upload and schedule](../curl/all-endpoints.md#3-upload-and-schedule)
- [Update](../curl/all-endpoints.md#4-update-a-post)
- [Webhook consumer](../curl/all-endpoints.md#5-verify-a-webhook)
- [Complete OpenAPI surface](https://docs.publora.com/openapi.yaml)
