# Core Workflows

This page is the canonical request-flow reference for Publora examples. Language and framework guides provide syntax and setup; the semantics live here. For the complete API surface, use the [OpenAPI document](https://docs.publora.com/openapi.yaml).

## Setup

```bash
export PUBLORA_API_KEY="sk_YOUR_API_KEY"
export PUBLORA_PLATFORM_ID="twitter-YOUR_CONNECTION_ID"
export PUBLORA_BASE_URL="https://api.publora.com/api/v1"
```

REST requests require `x-publora-key`. Platform IDs come from [List Platform Connections](../../endpoints/platform-connections.md).

## 1. Create a draft

Omitting `scheduledTime` intentionally creates a draft. A draft does not publish.

```bash
curl -sS -X POST "$PUBLORA_BASE_URL/create-post" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"content\":\"Draft from Publora\",\"platforms\":[\"$PUBLORA_PLATFORM_ID\"]}"
```

The response contains `success`, `postGroupId`, and `scheduledTime` (`null` for this draft). Save the 24-character hexadecimal `postGroupId` for later operations.

## 2. One-shot schedule

`scheduledTime` must be a future ISO 8601 timestamp. This runnable example schedules five minutes ahead:

```bash
FUTURE_TIME=$(date -u -v+5M +%Y-%m-%dT%H:%M:%S.000Z 2>/dev/null || date -u -d '+5 minutes' +%Y-%m-%dT%H:%M:%S.000Z)

curl -sS -X POST "$PUBLORA_BASE_URL/create-post" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: schedule-$(date +%s)" \
  -d "{\"content\":\"Scheduled from Publora\",\"platforms\":[\"$PUBLORA_PLATFORM_ID\"],\"scheduledTime\":\"$FUTURE_TIME\"}"
```

Use a stable `Idempotency-Key` when retrying the same logical request. Do not generate a new key for each retry.

## 3. Upload and schedule

The presigned-upload flow is `create draft` → `get-upload-url` → upload bytes → `complete-media` → `update-post` to schedule. Scheduling last avoids the media-attachment demotion behavior.

```bash
# Create a draft and copy postGroupId from the response.
POST_GROUP_ID="64f1a2b3c4d5e6f7890abcde"

# Request an upload URL. Copy uploadUrl and mediaId from the response.
curl -sS -X POST "$PUBLORA_BASE_URL/get-upload-url" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"fileName\":\"photo.jpg\",\"contentType\":\"image/jpeg\",\"type\":\"image\",\"postGroupId\":\"$POST_GROUP_ID\"}"

UPLOAD_URL="<UPLOAD_URL_FROM_RESPONSE>"
MEDIA_FILE_ID="<MEDIA_ID_FROM_RESPONSE>"

curl -sS -X PUT "$UPLOAD_URL" -H "Content-Type: image/jpeg" --data-binary @photo.jpg

# Probe/finalize the uploaded object before scheduling.
curl -sS -X POST "$PUBLORA_BASE_URL/complete-media/$MEDIA_FILE_ID" \
  -H "x-publora-key: $PUBLORA_API_KEY"

FUTURE_TIME=$(date -u -v+5M +%Y-%m-%dT%H:%M:%S.000Z 2>/dev/null || date -u -d '+5 minutes' +%Y-%m-%dT%H:%M:%S.000Z)
curl -sS -X PUT "$PUBLORA_BASE_URL/update-post/$POST_GROUP_ID" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: media-schedule-$POST_GROUP_ID" \
  -d "{\"status\":\"scheduled\",\"scheduledTime\":\"$FUTURE_TIME\"}"
```

For public HTTPS assets, `mediaUrls` on create/update is a shorter ingestion path. See [Media Uploads](../../guides/media-uploads.md) for its limits and append semantics.

## 4. Update a post

Update accepts at least one of `status`, `scheduledTime`, `content`, `platforms`, `platformSettings`, or `mediaUrls`. Omitted fields keep their stored value. `mediaUrls` appends media rather than replacing existing items; `platforms` does the opposite — it replaces the whole target set.

```bash
POST_GROUP_ID="64f1a2b3c4d5e6f7890abcde"
FUTURE_TIME=$(date -u -v+10M +%Y-%m-%dT%H:%M:%S.000Z 2>/dev/null || date -u -d '+10 minutes' +%Y-%m-%dT%H:%M:%S.000Z)

curl -sS -X PUT "$PUBLORA_BASE_URL/update-post/$POST_GROUP_ID" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: reschedule-$POST_GROUP_ID" \
  -d "{\"status\":\"scheduled\",\"scheduledTime\":\"$FUTURE_TIME\"}"
```

Correct the text and the target accounts of a draft or scheduled post. Copy the connection IDs
verbatim from `GET /platform-connections` — the array replaces the stored one:

```bash
curl -sS -X PUT "$PUBLORA_BASE_URL/update-post/$POST_GROUP_ID" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: edit-$POST_GROUP_ID" \
  -d '{"content":"Corrected launch announcement.","platforms":["linkedin-ABC123","threads-DEF456"]}'
```

See [Update Post](../../endpoints/update-post.md) for validation and state behavior.

## 5. Verify a webhook

Publora signs the exact request-body bytes with HMAC-SHA256. Verify the raw bytes before parsing JSON.

```javascript
import crypto from 'node:crypto';
import express from 'express';

const app = express();
const secret = process.env.PUBLORA_WEBHOOK_SECRET;

app.post('/publora-webhook', express.raw({ type: 'application/json' }), (req, res) => {
  const received = req.get('x-publora-signature') || '';
  const expected = crypto.createHmac('sha256', secret).update(req.body).digest('hex');
  const valid = received.length === expected.length &&
    crypto.timingSafeEqual(Buffer.from(received), Buffer.from(expected));

  if (!valid) return res.sendStatus(401);
  const envelope = JSON.parse(req.body.toString('utf8'));
  console.log(envelope.event, envelope.data);
  return res.sendStatus(200);
});
```

Delivery is a single attempt per event, not a retry queue. Respond quickly with `2xx`; see [Webhooks](../../endpoints/webhooks.md) for events, failure counting, and re-enabling disabled webhooks.

## Related capabilities

The five workflows above are the shared example contract, not the entire API. Use the endpoint pages and [OpenAPI](https://docs.publora.com/openapi.yaml) for connections, listing and deleting posts, platform limits, post logs, LinkedIn engagement and analytics, workspace operations, and webhook administration.
