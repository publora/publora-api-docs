# TypeScript Quick Start

TypeScript syntax for the five [Core Workflows](../curl/all-endpoints.md). Keep generated or application-specific types local; the canonical workflow page owns the wire contract.

## Typed client

```ts
type CreatePostResponse = {
  success: boolean;
  postGroupId: string;
  scheduledTime: string | null;
  warnings?: Array<{ code: string; message: string }>;
};

const baseUrl = 'https://api.publora.com/api/v1';

async function publora<T>(
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
  });
  const body: unknown = await response.json();
  if (!response.ok) throw new Error(`Publora returned ${response.status}`);
  return body as T;
}

const platformId = process.env.PUBLORA_PLATFORM_ID!;
```

## 1. Draft

```ts
const draft = await publora<CreatePostResponse>('create-post', {
  method: 'POST',
  body: JSON.stringify({ content: 'Draft from TypeScript', platforms: [platformId] }),
});
```

No `scheduledTime` means the post remains a draft.

## 2. One-shot schedule

```ts
const scheduled = await publora<CreatePostResponse>('create-post', {
  method: 'POST',
  body: JSON.stringify({
    content: 'Scheduled from TypeScript',
    platforms: [platformId],
    scheduledTime: new Date(Date.now() + 5 * 60_000).toISOString(),
  }),
}, crypto.randomUUID());
```

## 3. Upload and schedule

Follow [create draft → presign → PUT → complete → schedule](../curl/all-endpoints.md#3-upload-and-schedule). The storage PUT does not use the Publora API header.

```ts
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'content-type': file.type },
  body: file,
});
await publora(`complete-media/${mediaFileId}`, { method: 'POST' });
await publora(`update-post/${postGroupId}`, {
  method: 'PUT',
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: new Date(Date.now() + 5 * 60_000).toISOString(),
  }),
}, `schedule-${postGroupId}`);
```

## 4. Update

```ts
await publora(`update-post/${postGroupId}`, {
  method: 'PUT',
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: new Date(Date.now() + 10 * 60_000).toISOString(),
  }),
}, `reschedule-${postGroupId}`);
```

## 5. Webhook consumer

```ts
import crypto from 'node:crypto';
import express from 'express';

const app = express();
app.post('/publora-webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const expected = crypto.createHmac('sha256', process.env.PUBLORA_WEBHOOK_SECRET!)
    .update(req.body).digest('hex');
  const received = req.get('x-publora-signature') ?? '';
  const valid = received.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(received), Buffer.from(expected));
  if (!valid) return res.sendStatus(401);
  const envelope: { event: string; data: unknown } = JSON.parse(req.body.toString('utf8'));
  return res.sendStatus(200);
});
```

LinkedIn analytics, batch orchestration, and platform settings remain available through their endpoint guides and the [complete OpenAPI surface](https://docs.publora.com/openapi.yaml); they are not redefined here.
