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
| `mediaUrls` | string[] | No | Up to **10** public **https** image/video URLs. Publora downloads them server-side and attaches them to the post **before** validation, so you can attach media *and* schedule in one call (pass together with `scheduledTime`). This is the one-shot alternative to the draft → `get-upload-url` → schedule flow. Ingestion is rate-limited to 60 URLs/hour. See [Posts with Media](#posts-with-media). |

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

> **Media-required platforms (Instagram, TikTok, YouTube, Pinterest):** these platforms require media on every post. `create-post` **validates this at the scheduling gate**: if `scheduledTime` is set (so the post is created as `scheduled`) and one of these platforms has no media, the call is **rejected up front with HTTP 400 `MEDIA_REQUIRED`** — the validator runs with an empty media list. It is *not* accepted-then-failed-at-publish. Two ways to satisfy it: (1) pass `mediaUrls` (public https URLs) in the **same** `create-post` call to attach and schedule in one shot, or (2) create a **draft** (omit `scheduledTime`), attach media, then schedule via [Update Post](update-post.md). Creating a draft with no media is always allowed (drafts skip validation). See [Validation](../guides/validation.md) for the per-platform media matrix.

> **Two supported media flows:**
>
> **A. One-shot with `mediaUrls`** (public https URLs — best for agents):

```
POST /create-post   → content + platforms + scheduledTime + mediaUrls[]  (attached + scheduled in one call)
```

> **B. Draft → upload → schedule** (for local files):

```
1. POST /create-post              → Omit scheduledTime (creates draft)
2. POST /get-upload-url           → Get pre-signed URL for media
3. PUT {uploadUrl}                → Upload file to S3
4. POST /complete-media/:mediaId  → (optional) finalize/validate the upload early
5. PUT /update-post/:postGroupId  → Set status="scheduled" + scheduledTime
```

> **Do not** attach media via `get-upload-url` to an *already-scheduled* post: attaching media demotes the post back to `draft` (`postGroupDemoted: true`) and you must re-schedule. See [Upload Media](upload-media.md).

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
    "title": "",
    "madeForKids": false,
    "playlist": {
      "id": "",
      "platformId": ""
    }
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

> **YouTube** also accepts `tags`, `categoryId`, `madeForKids`, and a `playlist` object (`{ id, platformId }`) in addition to `privacy`/`title`. A `playlist` can be set directly on `create-post`. A custom `thumbnail` **cannot** be set on `create-post` — the thumbnail upload requires a `postGroupId`, so create the post first and set the thumbnail via `update-post`. See [YouTube → Platform-Specific Settings](../platforms/youtube.md#platform-specific-settings) for the full field reference.

> **Instagram** also accepts `coverUrl` (alias: `cover_url`) — a custom cover image for Reels. Provide a **publicly accessible http(s) URL to a JPEG image** (Instagram fetches it server-side at publish time), or [upload a cover file](upload-instagram-cover.md) (JPEG/PNG/WebP up to 8 MB) and use the URL it returns. Non-JPEG or non-http(s) URLs are rejected with `400`. An empty string clears the custom cover. Ignored for Stories and image posts. See [Instagram → Platform-Specific Settings](../platforms/instagram.md#platform-specific-settings).

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

### Post to all 10 platforms at once

> **Note:** This example shows the platform-ID format and fan-out only. As written it is **text-only** with `scheduledTime` set, so it is **rejected with HTTP 400 `MEDIA_REQUIRED`** — the **Instagram, TikTok, YouTube, and Pinterest** targets require media. To include them, either pass `mediaUrls` in this same call, or create as a draft (omit `scheduledTime`), [upload media](upload-media.md), then schedule.

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
      'telegram-000'
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
| 400 | `"mediaUrls must be an array of public https URLs"` | `mediaUrls` is present but not an array |
| 400 | `"mediaUrls supports at most 10 URLs per request"` | More than 10 URLs supplied (also: `"mediaUrls must contain at least one URL when provided"`, `"Every mediaUrls entry must be a non-empty string"`) |
| 400 | `"Media ingestion failed"` | One or more `mediaUrls` could not be downloaded/probed; per-URL detail in the `mediaResults` array |
| 429 | `"Media URL ingestion rate limit reached (60 URLs/hour)…"` | `code: "MEDIA_URL_RATE_LIMITED"`, `retryAfterSec` + `Retry-After` header |
| 400 | `"platformSettings must be an object"` | `platformSettings` is provided but is not a plain object (e.g., array, null, or primitive after JSON parsing) |
| 400 | `"Invalid x-publora-user-id"` | The `x-publora-user-id` header value is not a valid ObjectId format |
| 401 | `"API key is required"` | Missing `x-publora-key` header |
| 401 | `"Invalid API key"` | `x-publora-key` value is incorrect or revoked |
| 401 | `"Invalid API key owner"` | API key exists but the associated workspace/user could not be resolved |
| 403 | `"API access is not enabled for this account"` | The account has `apiAccess` disabled (a custom plan; standard plans including the free Starter plan include API access) |
| 403 | `"MCP access is not enabled for this account"` | Returned when `x-publora-client: mcp` is set but MCP access is not enabled |
| 403 | `"Workspace access is not enabled for this key"` | The API key does not have workspace/managed-user permissions |
| 403 | `"User is not managed by key"` | The `x-publora-user-id` references a user not managed by this API key |
| 403 | LimitExceededError (structured) | Plan limit reached (see below) |
| 500 | `"Failed to create post group"` | Unexpected server error |

> **Media-required platforms (Instagram, TikTok, YouTube, Pinterest):** when `scheduledTime` is set, `create-post` validates media presence and **rejects a media-less post with HTTP 400 `MEDIA_REQUIRED`** (the validator runs with an empty media list). Attach media first — pass `mediaUrls`, or use the draft flow. See [Posts with Media](#posts-with-media) and [Validation](../guides/validation.md). The 400 body shape:
>
> ```json
> {
>   "error": "Validation failed",
>   "validation": {
>     "valid": false,
>     "errors": [
>       {
>         "code": "MEDIA_REQUIRED",
>         "message": "Instagram posts require media",
>         "platform": "instagram",
>         "field": "media",
>         "suggestions": [
>           "Upload an image or video for this platform",
>           "This platform requires media. One-shot fix: POST /api/v1/create-post with mediaUrls (array of public https URLs) plus scheduledTime. Manual flow: create a draft (no scheduledTime), POST /api/v1/get-upload-url once per file, PUT the bytes with the same Content-Type, POST /api/v1/complete-media/:mediaId, then PUT /api/v1/update-post/:postGroupId with status='scheduled'."
>         ],
>         "severity": "error"
>       }
>     ],
>     "warnings": [],
>     "summary": { "errorCount": 1, "warningCount": 0, "affectedPlatforms": ["instagram"] }
>   }
> }
> ```
> (The second `suggestions` entry is the channel-aware recovery hint — MCP callers get the `create_post`/`get_upload_url` tool-name variant.)

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
