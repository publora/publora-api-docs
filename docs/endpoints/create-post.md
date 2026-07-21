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
| `Idempotency-Key` | No | Opt-in duplicate protection. A client-generated unique string. Retrying with the **same key and same body** replays the original response instead of creating a second post. See [Idempotency](#idempotency). |
| `Content-Type` | Yes | `application/json` |

> **Browser usage:** CORS permits `x-publora-key`, `x-publora-user-id`, `x-publora-client`, and `Idempotency-Key`, but only requests from origins in Publora's deployment allowlist are accepted. An arbitrary integrator origin can still fail preflight. Even from an allowed origin, embedding a secret API key in client-side code exposes it; prefer a server-side proxy.

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Conditional | Normally required and non-empty. It may be omitted or empty when a targeted LinkedIn connection has repost intent through `platformSettings.linkedin.repostEnabled: true` or a non-empty `repostParentUrn`; the repost settings must still form a valid parent/enable combination. Non-string truthy values (numbers, objects) can pass the initial content gate but cause unexpected behavior. |
| `platforms` | string[] | Yes | Array of `<lowercase-prefix>-<id>` connection IDs. The ID cannot contain whitespace, `/`, `?`, or `#`; colons are allowed (e.g., `twitter-123456789`, `bluesky-did:plc:abc123`). Each ID must appear at most once — a repeated ID is rejected with `400 "Platforms must not contain duplicates"`. |
| `scheduledTime` | string | No | ISO 8601 UTC datetime. If omitted, the post is created as a `draft`. A time in the past is **never silently accepted** — it is either clamped to server time with a `SCHEDULED_TIME_COERCED` warning in the response, or rejected with `400 SCHEDULED_TIME_IN_PAST`. See [Past scheduled times](#past-scheduled-times). |
| `platformSettings` | object | No | Per-platform settings that are merged with server-side defaults. User-provided values override defaults on a per-platform basis. The accepted keys are `tiktok`, `instagram`, `youtube`, `threads`, `telegram`, and `linkedin`. **Any unknown top-level platform or unknown nested key is rejected with `400 PLATFORM_SETTING_UNKNOWN` and nothing is persisted** — see [Unknown platformSettings paths](#unknown-platformsettings-paths) for the exact accepted tree. Each platform key must map to a plain object. Validation errors (`"Invalid platformSettings JSON"`, `"platformSettings must be an object"`) are returned if the field is present and malformed. |
| `mediaUrls` | string[] | No | Up to **10** public **https** image/video URLs. Publora downloads them server-side and attaches them to the post **before** validation, so you can attach media *and* schedule in one call (pass together with `scheduledTime`). This is the one-shot alternative to the draft → `get-upload-url` → schedule flow. Ingestion is rate-limited to 60 URLs/hour. See [Posts with Media](#posts-with-media). |

## Response

Returns HTTP **200** on success (not 201).

```json
{
  "success": true,
  "postGroupId": "507f1f77bcf86cd799439011",
  "scheduledTime": "2026-03-01T14:00:00.000Z"
}
```

Use the `postGroupId` to track, update, or delete the post.

| Field | Always present | Description |
|-------|----------------|-------------|
| `success` | Yes | `true` on a 200 |
| `postGroupId` | Yes | Post group ID — use it to track, update, or delete the post |
| `scheduledTime` | Yes | The **effective** scheduled time actually stored (ISO 8601 UTC), or `null` for a draft. Trust this over the value you sent — a permitted past-time clamp may change it; a plan-horizon violation is rejected rather than adjusted. |
| `warnings` | No | Present only when the request was accepted with a caveat. Array of `{ code, message, requested, effective }`. |
| `media` | No | Per-URL ingestion results, present only when `mediaUrls` was supplied |

When a past `scheduledTime` is clamped, the response carries a warning and `scheduledTime` reflects the clamped value:

```json
{
  "success": true,
  "postGroupId": "507f1f77bcf86cd799439011",
  "scheduledTime": "2026-03-01T14:03:11.000Z",
  "warnings": [
    {
      "code": "SCHEDULED_TIME_COERCED",
      "message": "Requested scheduled time 2026-03-01T14:01:00.000Z was in the past and was changed to server time 2026-03-01T14:03:11.000Z.",
      "requested": "2026-03-01T14:01:00.000Z",
      "effective": "2026-03-01T14:03:11.000Z"
    }
  ]
}
```

## Past scheduled times

A `scheduledTime` in the past is handled by how far in the past it is. The 5-minute
tolerance absorbs ordinary client clock skew; anything beyond it is a real bug in the
caller's logic.

| How far in the past | Behaviour today | Behaviour when strict mode is active (scheduled 2026-08-25 by default) |
|---------------------|-----------------|----------------------------|
| Less than 5 minutes | Clamped to server time + `SCHEDULED_TIME_COERCED` warning | **Unchanged** — clamped + warned |
| 5 minutes or more | Clamped to server time + `SCHEDULED_TIME_COERCED` warning | **Rejected** with `400` / `SCHEDULED_TIME_IN_PAST` |

> **⚠️ Scheduled strict date — 2026-08-25.** Sending a `scheduledTime` 5+ minutes in the
> past currently succeeds with a warning. It is scheduled to become a hard `400` on
> **2026-08-25**, unless production configuration overrides that date either way.
> If you see `SCHEDULED_TIME_COERCED` in your responses today, fix it before that date:
> the usual causes are an unsynced client clock, a local-time value sent without a UTC
> offset, or replaying a stored `scheduledTime` from an earlier run. Compare your
> requested time against the `serverTime` / `effective` values the API returns.

> **The 5-minute tolerance is permanent.** The sunset changes the handling of times
> **5+ minutes** in the past and nothing else. A time 2 minutes in the past will still
> be clamped-and-warned after 2026-08-25, exactly as it is today.

The rejection body:

```json
{
  "error": "Scheduled time is in the past. Server time is 2026-03-01T14:03:11.000Z UTC.",
  "code": "SCHEDULED_TIME_IN_PAST",
  "serverTime": "2026-03-01T14:03:11.000Z"
}
```

Use `serverTime` to measure your clock offset against Publora's.

## Unknown platformSettings paths

`platformSettings` is validated against a strict allowlist **before anything is
persisted**. An unrecognized top-level platform (e.g. `twitter`, `facebook`) or an
unrecognized nested key returns `400` and creates no post:

```json
{
  "error": "Unknown platformSettings path: youtube.thumbnail.mediaID",
  "code": "PLATFORM_SETTING_UNKNOWN",
  "field": "youtube.thumbnail.mediaID"
}
```

> **Why this matters:** a typo like `youtube.thumbnail.mediaID` (capital `D`) or
> `instagram.coverurl` previously passed silently and *wiped* the corresponding
> setting. It is now a loud `400` naming the exact `field`.

Nested reference objects are checked to their leaves — `youtube.playlist` and
`youtube.thumbnail` cannot carry extra keys. The complete accepted tree:

| Platform | Accepted keys |
|----------|---------------|
| `tiktok` | `viewerSetting`, `allowComments`, `allowDuet`, `allowStitch`, `commercialContent`, `brandOrganic`, `brandedContent` |
| `instagram` | `videoType`, `shareToFeed`, `coverUrl` (alias `cover_url`) |
| `youtube` | `privacy`, `title`, `tags`, `madeForKids`, `categoryId`, `playlist.{id,platformId}`, `thumbnail.{mediaId,id,url,path}` |
| `threads` | `replyControl` |
| `telegram` | `disableNotification`, `disableWebPagePreview`, `protectContent` |
| `linkedin` | `repostEnabled`, `repostParentUrn`, `repostVisibility` |

LinkedIn repost settings default to `false`, an empty parent URN, and an empty visibility. When enabled, use a canonical `urn:li:share:...` or `urn:li:ugcPost:...` parent. `CONNECTIONS` visibility is supported only for personal connections; scheduling a company-page repost with it returns `400` with `"LinkedIn company-page reposts cannot use CONNECTIONS visibility; choose PUBLIC"`. See [LinkedIn Repost Settings](#linkedin-repost-settings) for the full contract (commentary, media rules, error strings).

Still accepted, unchanged:

- **REST aliases** — `instagram.cover_url`, `youtube.thumbnail.id` (for `mediaId`),
  `youtube.thumbnail.path` (for `url`).
- **String booleans** — `"false"` / `"0"` / `"off"` coerce to `false` on `telegram` and
  `tiktok` boolean keys.
- **Comma-separated `youtube.tags`** — a string is split into an array.
- **Legacy flat fields** — `youtube.playlistId` and `youtube.thumbnailUrl` are not
  `PLATFORM_SETTING_UNKNOWN`; they keep returning their existing migration-hint `400`
  (e.g. *"playlistId is not supported; use …playlist"*). They are never persisted.

## Idempotency

Send an optional `Idempotency-Key` request header to make `create-post` safe to retry.
This solves the classic agent failure: **the request times out or the connection drops,
the client retries, and the user ends up with two identical posts**. With a key, the
retry replays the original response — one post, not two.

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Idempotency-Key: 8f1c0b6e-6a2f-4a1e-9f3b-2b1d0c9a7e55" \
  -d '{
    "content": "Excited to share our new product launch! 🚀 #launch",
    "platforms": ["twitter-123456789"],
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
```

| Situation | Result |
|-----------|--------|
| Key not seen before | Request runs normally; the response is recorded against the key |
| Same key + **same** body | The original response is **replayed** verbatim (same status, same `postGroupId`). No second post. |
| Same key + **different** body | `422` / `IDEMPOTENCY_KEY_CONFLICT` |
| Same key, first request **still running** | `409` / `IDEMPOTENCY_IN_FLIGHT` — retry shortly |

> **Opt-in.** Without the header, behaviour is exactly as before — no dedup, no new
> failure modes.

**Client guidance:**

- Generate a **fresh key per logical create** (e.g. `crypto.randomUUID()`), and reuse
  that same key for *every retry of that one create* — that is the entire point.
- Never reuse a key across different posts; the second one gets `422`.
- On `409`, wait and retry the identical request rather than dropping the key — a stuck
  in-flight claim frees up after ~3 minutes and the retry then completes or replays.
- Keys are scoped to the acting user (the managed user when `x-publora-user-id` is set),
  so keys cannot collide across accounts.
- **Records expire after 24 hours.** After that a reused key is treated as new and
  creates a second post — keep retries well inside that window.
- Body comparison is order-insensitive (JSON keys are canonicalized), so re-serializing
  the same payload does not trip a conflict. Extremely deeply nested bodies (>200 levels)
  are rejected with `400` / `IDEMPOTENCY_BODY_TOO_COMPLEX`.

## Posts with Media

> **Publishable media-required platforms (Instagram, TikTok, YouTube):** these platforms require media on every post. `create-post` **validates this at the scheduling gate**: if `scheduledTime` is set (so the post is created as `scheduled`) and one of these platforms has no media, the call is **rejected up front with HTTP 400 `MEDIA_REQUIRED`** — the validator runs with an empty media list. It is *not* accepted-then-failed-at-publish. Two ways to satisfy it: (1) pass `mediaUrls` (public https URLs) in the **same** `create-post` call to attach and schedule in one shot, or (2) create a **draft** (omit `scheduledTime`), attach media, then schedule via [Update Post](update-post.md). Creating a draft with no media is always allowed (drafts skip validation). Pinterest is connect-only: its registry limits and media validation do not imply publishing support. See [Validation](../guides/validation.md) for the per-platform media matrix.

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

> **Note:** Only `tiktok`, `instagram`, `youtube`, `threads`, `telegram`, and `linkedin` keys are recognized in `platformSettings`. Other platform keys (e.g., `twitter`, `facebook`, `bluesky`, `mastodon`) are **rejected with `400` / `PLATFORM_SETTING_UNKNOWN`** — they are no longer silently dropped. LinkedIn accepts `repostEnabled`, `repostParentUrn`, and `repostVisibility`. For `telegram`, only the three boolean keys shown above (`disableNotification`, `disableWebPagePreview`, `protectContent`) are accepted; any other key inside the telegram object is likewise a `400`, and string values like `"false"` / `"0"` / `"off"` are still coerced to `false`. See [Unknown platformSettings paths](#unknown-platformsettings-paths).

> **YouTube** also accepts `tags`, `categoryId`, `madeForKids`, and a `playlist` object (`{ id, platformId }`) in addition to `privacy`/`title`. A `playlist` can be set directly on `create-post`. Nested objects are validated to their leaves: `playlist` accepts only `id`/`platformId` and `thumbnail` only `mediaId`/`id`/`url`/`path` — any other nested key (e.g. `youtube.thumbnail.mediaID`) returns `400` / `PLATFORM_SETTING_UNKNOWN` rather than silently clearing the value. A custom `thumbnail` **cannot** be set on `create-post` — the thumbnail upload requires a `postGroupId`, so create the post first and set the thumbnail via `update-post`. See [YouTube → Platform-Specific Settings](../platforms/youtube.md#platform-specific-settings) for the full field reference.

> **Instagram** also accepts `coverUrl` (alias: `cover_url`) — a custom cover image for Reels. Provide a **publicly accessible http(s) URL to a JPEG image** (Instagram fetches it server-side at publish time), or [upload a cover file](upload-instagram-cover.md) (JPEG/PNG/WebP up to 8 MB) and use the URL it returns. Non-JPEG or non-http(s) URLs are rejected with `400`. An empty string clears the custom cover. Ignored for Stories and image posts. See [Instagram → Platform-Specific Settings](../platforms/instagram.md#platform-specific-settings).

## LinkedIn Repost Settings

Set `platformSettings.linkedin.repostParentUrn` to schedule a **repost** (reshare) of an existing LinkedIn post instead of a regular post. The group's `content` becomes the reshare commentary shown above the reposted post (the normal LinkedIn 3,000-character content limit applies); empty content is allowed for a plain repost.

```json
{
  "content": "My commentary on this great post.",
  "platforms": ["linkedin-Tz9W5i6ZYG"],
  "scheduledTime": "2026-08-01T14:00:00.000Z",
  "platformSettings": {
    "linkedin": {
      "repostParentUrn": "urn:li:share:7123456789012345678",
      "repostVisibility": "PUBLIC"
    }
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `repostParentUrn` | string | URN of the post to reshare — `urn:li:share:<id>` or `urn:li:ugcPost:<id>` (**not** `urn:li:activity:<id>`). Empty string = normal post (repost off) |
| `repostVisibility` | string | `PUBLIC` (default) or `CONNECTIONS`. Lowercase input is normalized to uppercase. `CONNECTIONS` is only valid for personal accounts — company (organization) pages are rejected at intake (see [the accepted tree](#unknown-platformsettings-paths)) and are always forced to `PUBLIC` at publish time |
| `repostEnabled` | boolean | Optional explicit toggle. `true` requires a non-empty `repostParentUrn`; `false` requires an empty one. When omitted, a non-empty `repostParentUrn` enables the repost |

**Rules:**

- **Media is forbidden on repost groups.** Scheduling a repost group with attached media fails validation with code `MEDIA_TYPE_NOT_SUPPORTED`: *"LinkedIn reposts cannot include media — remove the attached files or turn off the repost setting"*.
- **Publish-time failures:** if the parent post was deleted, made private, or is not reshareable, the scheduled post fails permanently with *"LinkedIn rejected the repost — the original post may have been deleted, made private, or is not reshareable."*

Field-level validation errors (400):

| Error | Cause |
|-------|-------|
| `"platformSettings.linkedin.repostParentUrn must be a LinkedIn post URN like urn:li:share:<id> or urn:li:ugcPost:<id>"` | Invalid URN shape (activity URNs are rejected) |
| `"platformSettings.linkedin.repostVisibility must be one of: PUBLIC, CONNECTIONS"` | Invalid visibility value |
| `"platformSettings.linkedin.repostParentUrn is required when repostEnabled is true"` | `repostEnabled: true` without a parent URN |
| `"platformSettings.linkedin.repostParentUrn must be empty when repostEnabled is false"` | `repostEnabled: false` with a parent URN set |

> **Tip:** To repost **immediately** without scheduling, use the dedicated [LinkedIn Reshare endpoint](linkedin-reshare.md) (`POST /linkedin-reshare`) instead.

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

> **Note:** This example shows the platform-ID format and fan-out only. As written it is **text-only** with `scheduledTime` set, so it is **rejected with HTTP 400 `MEDIA_REQUIRED`** — the **Instagram, TikTok, and YouTube** targets require media. To include them, either pass `mediaUrls` in this same call, or create as a draft (omit `scheduledTime`), [upload media](upload-media.md), then schedule. Pinterest is connect-only and is intentionally absent: satisfying its registry media validation does not make it publishable.

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

Platform IDs use `<lowercase-prefix>-<id>`. The prefix must contain only lowercase ASCII letters. The ID must be non-empty and cannot contain whitespace, `/`, `?`, or `#`; other characters, including colons, are allowed. For example, `bluesky-did:plc:abc123` is a valid target.

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Content is required"` | Missing `content` field or content is empty/whitespace-only |
| 400 | `"Platforms are required"` | `platforms` field is missing or null |
| 400 | `"At least one platform is required"` | `platforms` is an empty array |
| 400 | `"Platforms must be an array"` | `platforms` is not an array |
| 400 | `"Invalid platforms JSON format"` | `platforms` contains malformed JSON |
| 400 | `"Invalid platform ID format: <id>"` | Prefix is not lowercase letters, or the ID is empty or contains whitespace, `/`, `?`, or `#` |
| 400 | `"Platforms must not contain duplicates"` | The same connection ID appears more than once in `platforms` |
| 400 | `"Invalid scheduled time format"` | `scheduledTime` is not a valid ISO 8601 datetime |
| 400 | `"Scheduled time is in the past. Server time is <ISO> UTC."` | `code: "SCHEDULED_TIME_IN_PAST"`, `serverTime: "<ISO>"`. `scheduledTime` is 5+ minutes in the past while strict mode is active; the default transition is scheduled for **2026-08-25** but configuration can override it. |
| 400 | `"Unknown platformSettings path: <path>"` | `code: "PLATFORM_SETTING_UNKNOWN"`, `field: "<exact.path>"`. Unknown platform or nested key; nothing is persisted. See [Unknown platformSettings paths](#unknown-platformsettings-paths) |
| 400 | `"platformSettings.linkedin.*"` validation errors | Invalid LinkedIn repost settings (bad URN shape, bad visibility, `repostEnabled`/`repostParentUrn` mismatch) — see [LinkedIn Repost Settings](#linkedin-repost-settings) |
| 400 | `"LinkedIn company-page reposts cannot use CONNECTIONS visibility; choose PUBLIC"` | `repostVisibility: "CONNECTIONS"` while the scheduled group targets a LinkedIn company-page connection |
| 400 | `"Invalid platformSettings JSON"` | `platformSettings` was provided as a string that could not be parsed as valid JSON |
| 400 | `"Idempotency-Key request body is too deeply nested"` | `code: "IDEMPOTENCY_BODY_TOO_COMPLEX"`. Body nested more than 200 levels deep while using `Idempotency-Key` |
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
| 409 | `"A request with this idempotency key is still in flight"` | `code: "IDEMPOTENCY_IN_FLIGHT"`. An earlier request with the same `Idempotency-Key` has not finished. Retry the identical request shortly. See [Idempotency](#idempotency) |
| 422 | `"Idempotency key was already used with a different request body"` | `code: "IDEMPOTENCY_KEY_CONFLICT"`. The same `Idempotency-Key` was reused with a different body. Generate a fresh key per logical create |
| 500 | `"Failed to create post group"` | Unexpected server error |

> **Publishable media-required platforms (Instagram, TikTok, YouTube):** when `scheduledTime` is set, `create-post` validates media presence and **rejects a media-less post with HTTP 400 `MEDIA_REQUIRED`** (the validator runs with an empty media list). Attach media first — pass `mediaUrls`, or use the draft flow. Pinterest is connect-only and cannot be published even if media validation passes. See [Posts with Media](#posts-with-media) and [Validation](../guides/validation.md). The 400 body shape:
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
| `limitScope` | `POST_LIMIT_REACHED` | `"account"` or `"connection"`, matching the plan's monthly-post scope |
| `blockedPlatforms` | `POST_LIMIT_REACHED` | Per-connection `{ platformSelection, used, remaining }` entries; present only for a connection-scoped monthly-post rejection |
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

> **Note:** `processingStatus` is internal and is not returned by get-post or list-posts.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
