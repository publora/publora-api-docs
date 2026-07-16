# Webhooks

Receive real-time post lifecycle notifications, including scheduled, published, failed, and media-demotion events. `token.expiring` is defined and subscribable but is not currently dispatched, so do not depend on it for token monitoring.

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/webhooks` | List all webhooks |
| POST | `/webhooks` | Create a webhook |
| PATCH | `/webhooks/:id` | Update a webhook |
| DELETE | `/webhooks/:id` | Delete a webhook |
| POST | `/webhooks/:id/regenerate-secret` | Regenerate signing secret |

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |

---

## List Webhooks

```
GET https://api.publora.com/api/v1/webhooks
```

### Response

```json
{
  "success": true,
  "webhooks": [
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
      "name": "Production Notifications",
      "url": "https://your-app.com/webhooks/publora",
      "events": ["post.published", "post.failed"],
      "isActive": true,
      "failureCount": 0,
      "lastTriggeredAt": "2026-02-22T14:30:00.000Z",
      "createdAt": "2026-02-20T10:00:00.000Z",
      "updatedAt": "2026-02-22T14:30:00.000Z",
      "__v": 0
    }
  ]
}
```

> **Note:** Both API and dashboard list responses may include `__v` (Mongoose version key). This field can be safely ignored.

---

## Create Webhook

```
POST https://api.publora.com/api/v1/webhooks
```

### Request Body

```json
{
  "name": "Production Notifications",
  "url": "https://your-app.com/webhooks/publora",
  "events": ["post.published", "post.failed", "post.demoted"]
}
```

### Response

```json
{
  "success": true,
  "webhook": {
    "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
    "name": "Production Notifications",
    "url": "https://your-app.com/webhooks/publora",
    "events": ["post.published", "post.failed", "post.demoted"],
    "secret": "a1b2c3d4e5f6...your-signing-secret...x9y0z1",
    "isActive": true,
    "createdAt": "2026-02-22T10:00:00.000Z"
  }
}
```

> **Note:** The Create Webhook endpoint returns HTTP **201 (Created)**, not 200.

> **Important:** The `secret` is only returned once when creating the webhook. Store it securely for signature verification.

### Available Events

| Event | Description |
|-------|-------------|
| `post.scheduled` | Post was scheduled |
| `post.published` | Post was successfully published |
| `post.failed` | Post failed to publish |
| `post.demoted` | A scheduled post was returned to draft after media changed |
| `token.expiring` | Defined and subscribable, but not currently dispatched; do not build flows that depend on it |

---

## Update Webhook

```
PATCH https://api.publora.com/api/v1/webhooks/:id
```

### Request Body

```json
{
  "name": "Updated Name",
  "url": "https://new-url.com/webhook",
  "events": ["post.failed"],
  "isActive": false
}
```

All fields are optional. Only provided fields will be updated.

> **Note:** The API uses truthy checks on `name`, `url`, and `events`. Passing an empty string `""` for any of these fields will be silently ignored (not treated as an update). Only non-empty values trigger updates.

> **Note:** The `isActive` field requires a strict boolean type (`typeof isActive === "boolean"`). Passing a string like `"false"` or `"true"` will be silently ignored — only literal `true` or `false` values are accepted.

### Response

```json
{
  "success": true,
  "webhook": {
    "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
    "name": "Updated Name",
    "url": "https://new-url.com/webhook",
    "events": ["post.failed"],
    "isActive": false,
    "updatedAt": "2026-02-22T15:00:00.000Z"
  }
}
```

---

## Delete Webhook

```
DELETE https://api.publora.com/api/v1/webhooks/:id
```

### Response

```json
{
  "success": true
}
```

---

## Regenerate Secret

```
POST https://api.publora.com/api/v1/webhooks/:id/regenerate-secret
```

### Response

```json
{
  "success": true,
  "secret": "new-secret-here..."
}
```

---

## Dashboard vs API Differences

Webhook management has two implementations: the **public API** (`/api/v1/webhooks`) and a **dashboard route** (`/webhooks`). They share the same underlying data but differ in several behaviors:

| Behavior | API (`/api/v1/webhooks`) | Dashboard (`/webhooks`) |
|----------|--------------------------|-------------------------|
| **Create error (invalid events)** | Returns a flat `{ "error": "Invalid events" }` — submitted event names are **not** echoed back (to avoid leaking admin-only event names to non-admin probes). | Error echoes the invalid event names: `"Invalid events: foo"` |
| **Update field checks** | Uses truthy checks on `name`/`url`/`events` — empty string `""` is silently ignored | Uses `!== undefined` checks — empty string is treated as a value |
| **`isActive` type check** | Requires strict boolean (`typeof isActive === "boolean"`) — strings like `"false"` are silently ignored | Uses `isActive !== undefined` — accepts any truthy/falsy value |
| **Re-enable webhook** | Sets `isActive: true` but does **not** reset `failureCount` | Sets `isActive: true` **and** resets `failureCount` to 0 |
| **List response** | Excludes `userId` and `secret` from response; does **not** sort by `createdAt`; may include `__v` (Mongoose version key) | Excludes only `secret` from response; sorts by `createdAt` descending |
| **URL validation error** | Returns `"URL must use HTTPS"` for non-HTTP/HTTPS protocols | Returns `"Only HTTP and HTTPS URLs are allowed"` for non-HTTP/HTTPS protocols |
| **Update response fields** | Update response omits `failureCount` and `lastTriggeredAt` | Update response includes `failureCount` and `lastTriggeredAt` |
| **`::1` / `.localhost` blocking** | Does **not** block `::1` (IPv6 loopback) or `.localhost` subdomains | Blocks both `::1` and `.localhost` subdomains |

> **Tip:** If you need to fully reset a webhook's failure state through the API, delete and recreate it. The dashboard UI handles this automatically.

---

## Webhook Payload

> **Note:** The webhook delivery system operates as a separate internal service. The behavior described in the Webhook Payload, Signature Verification, and Webhook Reliability sections below reflects the production implementation.

When an event occurs, Publora sends a POST request to your webhook URL:

### Headers

| Header | Description |
|--------|-------------|
| `Content-Type` | `application/json` |
| `X-Publora-Signature` | HMAC-SHA256 signature of the payload |
| `X-Publora-Event` | Event type (e.g., `post.published`) |

### Payload Structure

```json
{
  "version": "1",
  "event": "post.published",
  "timestamp": "2026-02-22T14:30:00.000Z",
  "data": {
    "postId": "507f1f77bcf86cd799439012",
    "postGroupId": "507f1f77bcf86cd799439011",
    "platform": "linkedin",
    "platformId": "ABC123",
    "postedId": "urn:li:share:7654321",
    "permalink": null,
    "publishedAt": "2026-02-22T14:30:00.000Z"
  }
}
```

`version` is currently the string `"1"`. The HMAC covers the raw bytes of this complete envelope: `{ version, event, timestamp, data }`.

### Event-Specific Data

#### post.scheduled

```json
{
  "postGroupId": "507f1f77bcf86cd799439011",
  "scheduledTime": "2026-02-23T09:00:00.000Z",
  "platforms": ["linkedin-ABC123", "twitter-123456789"]
}
```

#### post.demoted

```json
{
  "postGroupId": "507f1f77bcf86cd799439011",
  "reason": "media_changed",
  "changeType": "attach",
  "mediaFileId": "507f1f77bcf86cd799439013"
}
```

`changeType` is `attach` or `detach`. The event is emitted when that media change demotes a scheduled group back to draft.

#### post.published

```json
{
  "postId": "507f1f77bcf86cd799439012",
  "postGroupId": "507f1f77bcf86cd799439011",
  "platform": "linkedin",
  "platformId": "ABC123",
  "postedId": "urn:li:share:7654321",
  "permalink": null,
  "publishedAt": "2026-02-22T14:30:00.000Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `postId` | string | Publora's ID for the individual platform post |
| `postGroupId` | string | Publora's ID for the parent post group |
| `platform` | string | Platform name (e.g., `linkedin`) |
| `platformId` | string/null | Raw platform account ID the post was published to — `null` when absent |
| `postedId` | string/null | **The platform's own ID for the live post** — use this to map the event to the real post on the platform |
| `permalink` | string/null | Public URL of the published post — `null` when unavailable |
| `publishedAt` | string | ISO-8601 timestamp of when the event was emitted |

> **Stable shape:** all seven keys are **always present** on `post.published`. Unavailable values are explicitly `null` rather than omitted, so no existence checks are needed. This holds on both delivery paths — the normal publish path and the recovered/reconciled publish path (where the scheduler re-emits a success it had already committed).

> **`permalink` is not yet populated.** The field is delivered on every `post.published` event, but is currently `null` for essentially all posts pending a separate backfill. If you need the live post today, resolve it from `platform` + `platformId` + `postedId`.

> **Publication identity:** `post.published` previously carried only Publora-internal IDs (`postId`, `postGroupId`), which gave receivers no way to locate the actual live post on the platform. `platformId`, `postedId` and `permalink` close that gap.

#### post.failed

```json
{
  "postId": "507f1f77bcf86cd799439012",
  "postGroupId": "507f1f77bcf86cd799439011",
  "platform": "threads",
  "error": {
    "code": "PLATFORM_AUTH_EXPIRED",
    "message": "Token expired"
  },
  "failedAt": "2026-02-22T14:30:00.000Z"
}
```

#### token.expiring

> **Planned shape:** this event is defined and can be selected when creating a webhook, but no production code currently dispatches it. Do not depend on receiving it.

```json
{
  "platform": "instagram",
  "platformId": "instagram-17841412345678",
  "username": "yourinstagram",
  "expiresAt": "2026-02-25T08:00:00.000Z"
}
```

---

## Signature Verification

Verify webhook authenticity using HMAC-SHA256:

### Node.js

```javascript
const crypto = require('crypto');

function verifyWebhookSignature(rawBody, signature, secret) {
  const expectedSignature = crypto
    .createHmac('sha256', secret)
    .update(rawBody)
    .digest('hex');

  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}

// Express middleware
app.post('/webhooks/publora', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-publora-signature'];
  const event = req.headers['x-publora-event'];

  if (!verifyWebhookSignature(req.body, signature, process.env.PUBLORA_WEBHOOK_SECRET)) {
    return res.status(401).json({ error: 'Invalid signature' });
  }

  const payload = JSON.parse(req.body.toString('utf8'));

  console.log(`Received ${event}:`, payload.data);

  // Handle the event
  switch (event) {
    case 'post.published':
      // Update your database, send notification, etc.
      break;
    case 'post.failed':
      // Alert your team, retry logic, etc.
      break;
  }

  res.status(200).json({ received: true });
});
```

### Python (Flask)

```python
import os
import hmac
import hashlib
from flask import Flask, request, jsonify

app = Flask(__name__)
WEBHOOK_SECRET = os.environ['PUBLORA_WEBHOOK_SECRET']

def verify_signature(raw_body, signature, secret):
    expected = hmac.new(
        secret.encode(),
        raw_body,
        hashlib.sha256
    ).hexdigest()
    return hmac.compare_digest(signature, expected)

@app.route('/webhooks/publora', methods=['POST'])
def handle_webhook():
    signature = request.headers.get('X-Publora-Signature')
    event = request.headers.get('X-Publora-Event')
    raw_body = request.get_data()

    if not verify_signature(raw_body, signature, WEBHOOK_SECRET):
        return jsonify({'error': 'Invalid signature'}), 401

    payload = request.get_json()

    print(f"Received {event}: {payload['data']}")

    if event == 'post.failed':
        # Alert team about failed post
        send_slack_alert(payload['data'])

    return jsonify({'received': True}), 200
```

---

## Webhook Reliability

- Webhooks timeout after 10 seconds
- Each event gets one delivery attempt; failed deliveries are not retried automatically
- After 5 consecutive failures, the webhook is automatically disabled
- A successful delivery resets `failureCount`; in a concurrent in-flight race it may also restore `isActive`. An already-disabled webhook receives no further deliveries and must be re-enabled explicitly with an update request.
- Re-enable a disabled webhook by updating `isActive: true`
- **Note:** Re-enabling a webhook via the API does **not** reset `failureCount`. The counter persists, meaning the webhook may be disabled again after fewer new failures. However, re-enabling a webhook via the **dashboard** does reset `failureCount` to 0. If you need to reset the counter through the API, delete and recreate the webhook.

> **Signature verification:** Compute the HMAC-SHA256 over the exact raw request body bytes. Capture and verify the raw body before parsing JSON; parsing and re-serializing can change the signed bytes.

## Limits

- Maximum 10 webhooks per user
- URL must use HTTPS (`"URL must use HTTPS"`) — note: despite the error message text, the API actually accepts both HTTP and HTTPS URLs
- Localhost URLs are blocked (`"Localhost URLs are not allowed"` — localhost, 127.0.0.1)
- Private IP addresses are blocked (`"Private IP addresses are not allowed"` — 10.x.x.x, 172.16-31.x.x, 192.168.x.x)
- Link-local addresses are blocked (`"Link-local addresses are not allowed"` — 169.254.x.x)
- Cloud metadata endpoints are blocked (`"Cloud metadata endpoints are not allowed"`)

> **Note:** The API route does not block `::1` (IPv6 loopback) or `.localhost` subdomains. These are only blocked by the dashboard route. This is a known difference — always use HTTP or HTTPS URLs with public hostnames.

---

## Examples

### Create Webhook with cURL

```bash
curl -X POST https://api.publora.com/api/v1/webhooks \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "My Webhook",
    "url": "https://my-app.com/webhooks/publora",
    "events": ["post.published", "post.failed"]
  }'
```

### List Webhooks

```bash
curl https://api.publora.com/api/v1/webhooks \
  -H "x-publora-key: YOUR_API_KEY"
```

### Delete Webhook

```bash
curl -X DELETE https://api.publora.com/api/v1/webhooks/65f8a1b2c3d4e5f6a7b8c9d0 \
  -H "x-publora-key: YOUR_API_KEY"
```

---

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Name, URL, and at least one event are required"` | Missing required fields |
| 400 | `"Invalid URL format"` | Malformed URL |
| 400 | `"Invalid events"` | One or more submitted event names is not in the allowed-events list for this caller. Names are **not** echoed back (admin-only events would otherwise leak to non-admin probes). Applies to both create and update. |
| 400 | `"URL must use HTTPS"` | URL uses unsupported protocol (note: both HTTP and HTTPS are actually accepted) |
| 400 | `"Localhost URLs are not allowed"` | URL points to localhost or 127.0.0.1 |
| 400 | `"Private IP addresses are not allowed"` | URL points to private network (10.x, 172.16-31.x, 192.168.x) |
| 400 | `"Link-local addresses are not allowed"` | URL points to 169.254.x.x |
| 400 | `"Cloud metadata endpoints are not allowed"` | URL targets cloud metadata service |
| 400 | `"Maximum of 10 webhooks per user"` | Webhook limit reached |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"Webhook not found"` | Invalid webhook ID |
| 500 | `"Failed to list webhooks"` | Server error listing webhooks |
| 500 | `"Failed to create webhook"` | Server error creating webhook |
| 500 | `"Failed to update webhook"` | Server error updating webhook |
| 500 | `"Failed to delete webhook"` | Server error deleting webhook |
| 500 | `"Failed to regenerate secret"` | Server error regenerating secret |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
