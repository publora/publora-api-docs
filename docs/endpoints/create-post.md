# Create Post

Schedule or immediately queue a post across multiple social media platforms.

## Endpoint

```
POST https://api.publora.com/api/v1/create-post
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Client identifier (e.g., `mcp` for MCP integrations). Affects which access controls are checked. |
| `Content-Type` | Yes | `application/json` |

> **Note:** The API does not include `x-publora-key` or `x-publora-user-id` in CORS allowed headers, so requests from browser-based clients will fail preflight checks. This API is designed for server-to-server use only.

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | Post text content (cannot be empty or whitespace-only). Must be a string — non-string truthy values (numbers, objects) will pass validation but cause unexpected behavior. |
| `platforms` | string[] | Yes | Array of platform connection IDs matching `/^[a-z]+-[a-zA-Z0-9_-]+$/` (e.g., `twitter-123456789`, `linkedin-ABC123`) |
| `scheduledTime` | string | No | ISO 8601 UTC datetime. If omitted, the post is created as a `draft`. If the scheduled time is in the past, it is silently set to the current time |
| `platformSettings` | object | No | Per-platform settings that are merged with server-side defaults. User-provided values override defaults on a per-platform basis. Only `tiktok`, `instagram`, `youtube`, `threads`, and `telegram` keys are accepted — any other platform keys are silently dropped. Each platform key must map to a plain object; non-object values are skipped. See [Default Platform Settings](#default-platform-settings) below for the full list of defaults and merge behavior. Validation errors (`"Invalid platformSettings JSON"`, `"platformSettings must be an object"`) are returned if the field is present and malformed. |

## Response

Returns HTTP **200** on success (not 201).

```json
{
  "success": true,
  "postGroupId": "507f1f77bcf86cd799439011"
}
```

Use the `postGroupId` to track, update, or delete the post.

## Posts with Media

> **Media-only platforms:** **Instagram, TikTok, YouTube, and Pinterest require media on every post** — a text-only post to these platforms cannot be published. `create-post` does **not** validate this (it only checks basic field presence), so a text-only post to these platforms is accepted here and then **fails when it is published** (Instagram, for example, errors with *"Instagram posts require at least one media item"*). When validation does run — the dashboard, or the validated update flow — it is rejected up front with the `MEDIA_REQUIRED` code instead. Always attach media via the draft → upload → schedule flow below so the post goes out cleanly. See [Validation](../guides/validation.md) for the per-platform media matrix.

> **Important:** When attaching images or videos, create the post as a **draft** (omit `scheduledTime`), upload media, then schedule via [Update Post](update-post.md).

```
1. POST /create-post              → Omit scheduledTime (creates draft)
2. POST /get-upload-url           → Get pre-signed URL for media
3. PUT {uploadUrl}                → Upload file to S3
4. PUT /update-post/:postGroupId  → Set scheduledTime to schedule
```

This ensures media is fully uploaded before the scheduler processes your post. See [Upload Media](upload-media.md) for details.

## Default Platform Settings

When creating via the API, these defaults are applied automatically. If you provide a `platformSettings` object in the request body, your values are **merged** with these defaults on a per-platform basis — user-provided fields override the corresponding default fields, while any default fields you omit are preserved. See [Default Platform Settings](#default-platform-settings) for the full list.

```json
{
  "tiktok": {
    "viewerSetting": "PUBLIC_TO_EVERYONE",
    "allowComments": true,
    "allowDuet": false,
    "allowStitch": false,
    "commercialContent": false,
    "brandOrganic": false,
    "brandedContent": false
  },
  "instagram": {
    "videoType": "REELS"
  },
  "youtube": {
    "privacy": "public",
    "title": ""
  },
  "threads": {
    "replyControl": ""
  },
  "telegram": {
    "disableNotification": false,
    "disableWebPagePreview": false,
    "protectContent": false
  }
}
```

> **Note:** Only `tiktok`, `instagram`, `youtube`, `threads`, and `telegram` keys are recognized in `platformSettings`. Other platform keys (e.g., `twitter`, `linkedin`, `facebook`, `bluesky`, `mastodon`) are silently ignored and dropped. For `telegram`, only the three boolean keys shown above (`disableNotification`, `disableWebPagePreview`, `protectContent`) are accepted; any other keys inside the telegram object are dropped, and string values like `"false"` / `"0"` / `"off"` are coerced to `false`.

## Examples

### Schedule a text post to X and LinkedIn

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Excited to share our new product launch! 🚀 #launch',
    platforms: ['twitter-123456789', 'linkedin-ABC123'],
    scheduledTime: '2026-03-01T14:00:00.000Z'
  })
});
const data = await response.json();
console.log(data.postGroupId);
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Excited to share our new product launch! 🚀 #launch',
        'platforms': ['twitter-123456789', 'linkedin-ABC123'],
        'scheduledTime': '2026-03-01T14:00:00.000Z'
    }
)
print(response.json()['postGroupId'])
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Excited to share our new product launch! 🚀 #launch",
    "platforms": ["twitter-123456789", "linkedin-ABC123"],
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
```

#### Node.js (axios)

```javascript
const axios = require('axios');

const { data } = await axios.post(
  'https://api.publora.com/api/v1/create-post',
  {
    content: 'Excited to share our new product launch! 🚀 #launch',
    platforms: ['twitter-123456789', 'linkedin-ABC123'],
    scheduledTime: '2026-03-01T14:00:00.000Z'
  },
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
console.log(data.postGroupId);
```

### Post to all 11 platforms at once

> **Note:** This example shows the platform-ID format and fan-out only. As written it is **text-only**, so the **Instagram, TikTok, YouTube, and Pinterest** targets would fail at publish time — those platforms require media. To publish to them, create this as a draft (omit `scheduledTime`), [upload media](upload-media.md), then schedule.

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Big announcement dropping tomorrow. Stay tuned.',
    platforms: [
      'twitter-111', 'linkedin-222', 'instagram-333',
      'threads-444', 'tiktok-555', 'youtube-666',
      'facebook-777', 'bluesky-888', 'mastodon-999',
      'telegram-000', 'pinterest-101'
    ],
    scheduledTime: '2026-03-01T09:00:00.000Z'
  })
});
```

### Create a draft (no scheduled time)

```python
response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Draft post -- will schedule later',
        'platforms': ['twitter-123456789']
    }
)
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Content is required"` | Missing `content` field or content is empty/whitespace-only |
| 400 | `"Platforms are required"` | `platforms` field is missing or null |
| 400 | `"At least one platform is required"` | `platforms` is an empty array |
| 400 | `"Platforms must be an array"` | `platforms` is not an array |
| 400 | `"Invalid platforms JSON format"` | `platforms` contains malformed JSON |
| 400 | `"Invalid platform ID format: <id>"` | Platform ID does not match `/^[a-z]+-[a-zA-Z0-9_-]+$/` |
| 400 | `"Invalid scheduled time format"` | `scheduledTime` is not a valid ISO 8601 datetime |
| 400 | `"Invalid platformSettings JSON"` | `platformSettings` was provided as a string that could not be parsed as valid JSON |
| 400 | `"platformSettings must be an object"` | `platformSettings` is provided but is not a plain object (e.g., array, null, or primitive after JSON parsing) |
| 400 | `"Invalid x-publora-user-id"` | The `x-publora-user-id` header value is not a valid ObjectId format |
| 401 | `"API key is required"` | Missing `x-publora-key` header |
| 401 | `"Invalid API key"` | `x-publora-key` value is incorrect or revoked |
| 401 | `"Invalid API key owner"` | API key exists but the associated workspace/user could not be resolved |
| 403 | `"API access is not enabled for this account"` | The account's plan does not include API access (e.g., Starter plan or custom plan with `apiAccess` disabled) |
| 403 | `"MCP access is not enabled for this account"` | Returned when `x-publora-client: mcp` is set but MCP access is not enabled |
| 403 | `"Workspace access is not enabled for this key"` | The API key does not have workspace/managed-user permissions |
| 403 | `"User is not managed by key"` | The `x-publora-user-id` references a user not managed by this API key |
| 403 | LimitExceededError (structured) | Plan limit reached (see below) |
| 500 | `"Failed to create post group"` | Unexpected server error |

> **Media-requiring platforms (Instagram, TikTok, YouTube, Pinterest):** `create-post` does not validate media presence, so a text-only post to these platforms is **not** rejected here — it fails later when published (e.g. Instagram: *"Instagram posts require at least one media item"*). The validated flow returns the `MEDIA_REQUIRED` code instead. See [Posts with Media](#posts-with-media) and [Validation](../guides/validation.md).

### LimitExceededError (403)

When a plan limit is exceeded, the API returns a structured error response. The `error` field contains a short label, while the `message` field contains the full human-readable explanation.

> **Note:** The first time a post limit is exceeded in a given billing month, an email notification is sent to the account owner. This notification is sent only once per month.

```json
{
  "error": "Post limit reached",
  "code": "POST_LIMIT_REACHED",
  "metric": "posts.platform_monthly",
  "message": "Monthly post limit reached. Your Pro plan allows 100 platform posts per month.",
  "limit": 100,
  "used": 100,
  "requested": 2,
  "remaining": 0,
  "periodStart": "2026-03-01T00:00:00.000Z",
  "periodEnd": "2026-04-01T00:00:00.000Z",
  "planName": "Pro"
}
```

> **Note:** Not all fields are present on every error code. For example, `SCHEDULE_HORIZON_REACHED` returns `null` for `used`, `requested`, `remaining`, `periodStart`, and `periodEnd`.

Some limit errors include additional context fields:

| Field | Present on | Description |
|-------|-----------|-------------|
| `scheduledTime` | `SCHEDULE_HORIZON_REACHED` | The requested scheduled time that exceeded the horizon |
| `maxScheduledDate` | `SCHEDULE_HORIZON_REACHED` | The furthest date allowed by the current plan |
| `scope` | `POST_LIMIT_REACHED`, `SCHEDULED_POST_LIMIT_REACHED` | Present when the limit is connection-scoped (e.g., `"connection"`). When connection-scoped, top-level `used` and `remaining` are `null` — per-connection values are in `blockedPlatforms` only. |
| `blockedPlatforms` | `POST_LIMIT_REACHED`, `SCHEDULED_POST_LIMIT_REACHED` | Array of objects with `platformSelection`, `used`, and `remaining` fields. Present when scope is connection-level |
| `overLimitBy` | `CONNECTIONS_OVER_LIMIT` | Number of connections over the plan limit |
| `disallowedPlatforms` | `PLATFORM_NOT_AVAILABLE` | Array of platform names not available on the current plan |
| `allowedPlatforms` | `PLATFORM_NOT_AVAILABLE` | Array of platform names available on the current plan |

Possible `code` values:

| Code | Error value | Description |
|------|-------------|-------------|
| `POST_LIMIT_REACHED` | `"Post limit reached"` | Monthly post limit exceeded |
| `SCHEDULED_POST_LIMIT_REACHED` | `"Scheduled post limit reached"` | Scheduled post limit exceeded |
| `SCHEDULE_HORIZON_REACHED` | `"Schedule horizon reached"` | Scheduling too far in the future for current plan |
| `CONNECTIONS_OVER_LIMIT` | `"Account over channel limit"` | Too many platform connections for current plan |
| `PLATFORM_NOT_AVAILABLE` | `"Platform not available"` | Platform not available on current plan (e.g., a custom plan with restricted `allowedPlatforms`). Metric: `posts.platform_monthly`. |

## Post Statuses

After creation, the post group goes through these states:

```
draft → scheduled → published
                  → failed
                  → partially_published
```

**Post group statuses** (returned in `status` field):
- **draft**: Saved but not scheduled
- **scheduled**: Will be published at `scheduledTime`
- **published**: Successfully posted on all platforms
- **failed**: Failed on all platforms
- **partially_published**: Succeeded on some, failed on others

> **Note:** Individual platform records (ScheduledPost) have their own statuses including `pending` and `processing`, which reflect per-platform delivery state. The post group `status` field above is a rollup and does not include `processing`. A separate `processingStatus` field on the post group tracks whether the post is currently being processed. Possible `processingStatus` values are: `pending`, `processing`, `finished`.

> **Note:** The `processingStatus` field is available in the [get-post](get-post.md) response but is **not** included in [list-posts](list-posts.md) responses.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
