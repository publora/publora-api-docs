# JavaScript Quick Start

JavaScript syntax for the five [Core Workflows](../curl/all-endpoints.md). That page owns request ordering, response semantics, and upload behavior.

## Setup

```javascript
const baseUrl = 'https://api.publora.com/api/v1';
const apiKey = process.env.PUBLORA_API_KEY;

async function publora(path, { idempotencyKey, ...init } = {}) {
  const response = await fetch(`${baseUrl}/${path}`, {
    ...init,
    headers: {
      'content-type': 'application/json',
      'x-publora-key': apiKey,
      ...(idempotencyKey ? { 'Idempotency-Key': idempotencyKey } : {}),
      ...init.headers,
    },
  });
  const body = await response.json();
  if (!response.ok) throw new Error(body.error ?? `HTTP ${response.status}`);
  return body;
}

const platformId = process.env.PUBLORA_PLATFORM_ID;
```

## 1. Draft

```javascript
const draft = await publora('create-post', {
  method: 'POST',
  body: JSON.stringify({ content: 'Draft from JavaScript', platforms: [platformId] }),
});
```

No `scheduledTime` means draft; it will not publish.

## 2. One-shot schedule

```javascript
const scheduledTime = new Date(Date.now() + 5 * 60_000).toISOString();
const scheduled = await publora('create-post', {
  method: 'POST',
  idempotencyKey: crypto.randomUUID(),
  body: JSON.stringify({ content: 'Scheduled from JavaScript', platforms: [platformId], scheduledTime }),
});
```

## 3. Upload and schedule

Follow the canonical [presigned upload sequence](../curl/all-endpoints.md#3-upload-and-schedule): create a draft, request `get-upload-url`, PUT the file bytes, call `complete-media/{mediaFileId}`, then schedule with `update-post`. Each API step uses the `publora` helper; upload bytes directly to the returned URL without `x-publora-key`.

```javascript
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'content-type': file.type },
  body: file,
});

await publora(`complete-media/${mediaFileId}`, { method: 'POST' });
await publora(`update-post/${postGroupId}`, {
  method: 'PUT',
  idempotencyKey: `schedule-${postGroupId}`,
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: new Date(Date.now() + 5 * 60_000).toISOString(),
  }),
});
```

## 4. Update

```javascript
await publora(`update-post/${postGroupId}`, {
  method: 'PUT',
  idempotencyKey: `reschedule-${postGroupId}`,
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: new Date(Date.now() + 10 * 60_000).toISOString(),
  }),
});
```

## 5. Webhook consumer

```javascript
import crypto from 'node:crypto';
import express from 'express';

const app = express();
app.post('/publora-webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const expected = crypto.createHmac('sha256', process.env.PUBLORA_WEBHOOK_SECRET)
    .update(req.body).digest('hex');
  const received = req.get('x-publora-signature') ?? '';
  const valid = received.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(received), Buffer.from(expected));
  if (!valid) return res.sendStatus(401);
  const envelope = JSON.parse(req.body.toString('utf8'));
  return res.sendStatus(200);
});
```

Verify raw bytes before parsing. See [Webhooks](../../endpoints/webhooks.md) and the [complete OpenAPI surface](https://docs.publora.com/openapi.yaml).
