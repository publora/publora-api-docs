# Update Post

Modify the scheduling time or status of an existing post. Updating a post group also updates all associated platform-specific posts.

## Endpoint

```
PUT https://api.publora.com/api/v1/update-post/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Client identifier (e.g., `"mcp"`) |
| `Content-Type` | Yes | `application/json` |
| `Idempotency-Key` | No | Opt-in retry safety. Any client-generated unique string (e.g. a UUID). See [Idempotency](#idempotency). **Strongly recommended when sending `mediaUrls`** — without it, a retried update appends the media a second time. |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID to update |

## Request Body

At least one of `status`, `scheduledTime`, `platformSettings`, or `mediaUrls` must be provided. Missing all four returns `400 { "error": "At least one of status, scheduledTime, platformSettings, or mediaUrls must be provided" }`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | `"draft"` or `"scheduled"` |
| `scheduledTime` | string | No | New ISO 8601 UTC datetime |
| `platformSettings` | object | No | Per-platform settings using the canonical [create-post allowlist](./create-post.md#unknown-platformsettings-paths), including LinkedIn repost settings. Merged with existing settings server-side — fields you omit are preserved. Any unknown top-level platform **or** unknown nested key is **rejected with `400 PLATFORM_SETTING_UNKNOWN`** and nothing is persisted — see [Unknown platformSettings paths](#unknown-platformsettings-paths). |
| `mediaUrls` | string[] | No | Public **`https://`** URLs (1–10 per request) that the server downloads and attaches to the post. **Semantics are APPEND** — the media is added to the post's existing media, never replacing it. Ingestion is all-or-nothing: if any URL fails, none are attached. Plain `http://`, embedded credentials, non-443 ports, and internal/loopback hosts are rejected. Because a retry re-appends, **send an [`Idempotency-Key`](#idempotency) whenever this field is present.** |

> **YouTube playlist & thumbnail (partial update):** `platformSettings` supports partial per-platform updates, so you can change `youtube.playlist` or set/clear `youtube.thumbnail` without resending the other YouTube fields. The **thumbnail can only be set here** (not on `create-post`, which has no `postGroupId` yet). Clear either by sending empty strings — `youtube.playlist` as `{ "id": "", "platformId": "" }`, or `youtube.thumbnail` as `{ "url": "" }`. Playlist or thumbnail changes return **`409`** while the post is publishing/processing (e.g. *"Post group is currently being published; cannot change YouTube playlist. Retry once publishing completes."*). See [YouTube → Platform-Specific Settings](../platforms/youtube.md#platform-specific-settings).

> **Instagram Reels cover:** set or change `instagram.coverUrl` (alias: `cover_url`) here — a JPEG image used as the Reel's cover. Either pass a publicly accessible http(s) URL, or [upload a cover file](upload-instagram-cover.md) (JPEG/PNG/WebP up to 8 MB) and set the returned URL. Send `{ "instagram": { "coverUrl": "" } }` to clear it (the cover falls back to automatic/frame-based selection). Non-JPEG or non-http(s) URLs are rejected with `400`; a change is rejected with `409` while the post is publishing. See [Instagram → Platform-Specific Settings](../platforms/instagram.md#platform-specific-settings).

> **LinkedIn repost (reshare):** `platformSettings.linkedin` turns the group into a repost of an existing LinkedIn post — set `repostParentUrn` to a `urn:li:share:<id>` or `urn:li:ugcPost:<id>` (send an empty string to turn the repost off and revert to a normal post), and optionally `repostVisibility` (`PUBLIC` default; `CONNECTIONS` is personal-accounts-only). The group's post content becomes the reshare commentary (3,000-character LinkedIn limit). **Media is forbidden on repost groups** — scheduling a repost group with attached media fails validation with code `MEDIA_TYPE_NOT_SUPPORTED` (*"LinkedIn reposts cannot include media — remove the attached files or turn off the repost setting"*). If the parent post was deleted, made private, or is not reshareable, the post fails permanently at publish time with *"LinkedIn rejected the repost — the original post may have been deleted, made private, or is not reshareable."* See [Create Post → LinkedIn Repost Settings](./create-post.md#linkedin-repost-settings) for the full field reference and validation error strings.

### Unknown platformSettings paths

Every path you send is checked against the accepted tree **before anything is written**. An unrecognized top-level platform, or an unrecognized nested key under a known one, is rejected — the whole request is a no-op:

```json
{
  "error": "Unknown platformSettings path: youtube.thumbnail.mediaID",
  "code": "PLATFORM_SETTING_UNKNOWN",
  "field": "youtube.thumbnail.mediaID"
}
```

`field` is the exact dotted path that failed, so you can point a user straight at the typo. Nested references are checked down to their leaves — `youtube.playlist.{id,platformId}` and `youtube.thumbnail.{mediaId,id,url,path}`. This matters most for thumbnails: a miscased `youtube.thumbnail.mediaID` previously slipped through the allowlist and **silently cleared an already-uploaded thumbnail**. It is now a `400`.

Values under a known leaf are never traversed, so these keep working unchanged:

- Aliases — `instagram.cover_url` (for `coverUrl`), `youtube.thumbnail.id` (for `mediaId`), `youtube.thumbnail.path` (for `url`)
- String booleans — `"true"` / `"false"` / `"1"` / `"0"` / `"yes"` / `"no"` / `"on"` / `"off"`
- Comma-separated `youtube.tags`

The deprecated flat fields `youtube.playlistId` and `youtube.thumbnailUrl` still return their existing guidance errors (e.g. *"playlistId is not supported; use …playlist"*) rather than `PLATFORM_SETTING_UNKNOWN`.

### ISO 8601 DateTime Format

The `scheduledTime` must be a valid ISO 8601 UTC datetime string:

```
YYYY-MM-DDTHH:mm:ss.sssZ
```

**Valid formats:**
```
2026-03-15T10:00:00.000Z    ✓ Full format with milliseconds
2026-03-15T10:00:00Z        ✓ Without milliseconds
2026-03-15T10:00:00+00:00   ✓ With explicit UTC offset
```

**Invalid formats:**
```
2026-03-15 10:00:00         ✗ Missing T separator and timezone
March 15, 2026              ✗ Not ISO 8601
03/15/2026                  ✗ Not ISO 8601
2026-03-15                  ✗ Missing time component
```

### Timing Constraints

- **Past times:** Never silent. A time in the past is either clamped to the current server time **with a `warnings` entry in the response**, or rejected with `400 SCHEDULED_TIME_IN_PAST` — see [Past scheduled times](#past-scheduled-times) for the exact rule and the migration date.
- **No scheduled time:** If the resulting status is `scheduled` and neither the request body nor the existing post has a `scheduledTime`, it defaults to the current time (`new Date()`). This default does not apply when updating to `draft` — the existing value is preserved
- **Maximum:** Recommended within 2 months for best reliability
- **Timezone:** Always use UTC (Z suffix or +00:00 offset)

### Past scheduled times

| How far in the past | Today | When strict mode is active (scheduled **2026-08-25** by default) |
|---|---|---|
| Less than 5 minutes | Clamped to server time + `SCHEDULED_TIME_COERCED` warning | **Unchanged — still clamped + warned** |
| 5 minutes or more | Clamped to server time + `SCHEDULED_TIME_COERCED` warning | **Rejected — `400 SCHEDULED_TIME_IN_PAST`** |

> **The 5-minute tolerance is permanent.** The *5-minutes-or-more* case is scheduled to flip from clamp-and-warn to a hard `400` on 2026-08-25, unless production configuration overrides that date either way.

Clamped requests still return `200`, and the response carries the warning:

```json
{
  "success": true,
  "message": "Post updated successfully",
  "scheduledTime": "2026-07-15T09:31:04.812Z",
  "warnings": [
    {
      "code": "SCHEDULED_TIME_COERCED",
      "message": "Requested scheduled time 2026-07-15T08:00:00.000Z was in the past and was changed to server time 2026-07-15T09:31:04.812Z.",
      "requested": "2026-07-15T08:00:00.000Z",
      "effective": "2026-07-15T09:31:04.812Z"
    }
  ],
  "postGroup": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "scheduled",
    "scheduledTime": "2026-07-15T09:31:04.812Z"
  }
}
```

Once strict mode is on, that same request returns:

```json
{
  "error": "Scheduled time is in the past. Server time is 2026-07-15T09:31:04.812Z UTC.",
  "code": "SCHEDULED_TIME_IN_PAST",
  "serverTime": "2026-07-15T09:31:04.812Z"
}
```

**Migrate now:** treat every `SCHEDULED_TIME_COERCED` warning where the gap is 5 minutes or more as a future `400`. Log it, and re-send a time in the future. `serverTime` on the error is the authoritative clock to compute against. For the full rationale and cross-endpoint behaviour, see [Scheduling → Past scheduled times](../guides/scheduling.md#past-scheduled-times).

> **Status-only transitions validate the *stored* time.** Sending `{ "status": "scheduled" }` with **no** `scheduledTime`, on a post that is not already scheduled, runs the post's **existing stored** time through the rules above. An old draft whose stored time has since passed therefore now returns a `SCHEDULED_TIME_COERCED` warning (and, once strict mode is on, a `400 SCHEDULED_TIME_IN_PAST`). Previously it silently published at the stale time — effectively immediately. If you flip old drafts to `scheduled`, always send an explicit fresh `scheduledTime` alongside the status.

### Scheduling Limits

When rescheduling a post (changing `scheduledTime`), the update flow performs two validations:

1. **Platform availability check** (`assertPlatformsAllowed`): Verifies that all platforms targeted by the post are still available on the user's current plan. If a platform is no longer allowed, a `PLATFORM_NOT_AVAILABLE` error is returned.
2. **Limits check**: Validates that the new time does not exceed the account's posting limits for the target time slot.

If either check fails, the request returns a **403** error with a `LimitExceededError` message describing which limit was hit.

> **Note:** There is no minimum-interval adjustment. A schedule-horizon violation returns `403`; only the documented past-time compatibility ramp can clamp a time, with a `SCHEDULED_TIME_COERCED` warning.

### Helper Functions

**JavaScript:**
```javascript
function toISO8601(date) {
  return new Date(date).toISOString();
}

function scheduleForTomorrow(hour = 9, minute = 0) {
  const date = new Date();
  date.setDate(date.getDate() + 1);
  date.setUTCHours(hour, minute, 0, 0);
  return date.toISOString();
}

// Usage
const scheduledTime = scheduleForTomorrow(14, 0); // Tomorrow at 2pm UTC
// "2026-02-21T14:00:00.000Z"
```

**Python:**
```python
from datetime import datetime, timedelta, timezone

def to_iso8601(dt):
    """Convert datetime to ISO 8601 UTC string."""
    if dt.tzinfo is None:
        dt = dt.replace(tzinfo=timezone.utc)
    return dt.astimezone(timezone.utc).strftime('%Y-%m-%dT%H:%M:%S.000Z')

def schedule_for_tomorrow(hour=9, minute=0):
    """Get ISO 8601 string for tomorrow at specified time (UTC)."""
    tomorrow = datetime.now(timezone.utc) + timedelta(days=1)
    scheduled = tomorrow.replace(hour=hour, minute=minute, second=0, microsecond=0)
    return to_iso8601(scheduled)

# Usage
scheduled_time = schedule_for_tomorrow(14, 0)  # Tomorrow at 2pm UTC
# "2026-02-21T14:00:00.000Z"
```

## Idempotency

Send an optional `Idempotency-Key` header with any value unique to the logical operation (a UUID works). It is **opt-in** — omit it and behaviour is exactly as before.

| Situation | Result |
|---|---|
| Same key, same body | The original response is **replayed** (same status code, same body). The update is not applied twice. |
| Same key, **different** body | `422` `IDEMPOTENCY_KEY_CONFLICT` |
| Same key, original still running | `409` `IDEMPOTENCY_IN_FLIGHT` |
| New key | Processed normally |

Keys are scoped to the **acting user** (the managed user when `x-publora-user-id` is set) and expire **24 hours** after first use. Two different users can safely use the same key string.

> **Why it matters most here:** `mediaUrls` on `update-post` has **append** semantics. A timeout or network blip on a request that already committed leaves you no way to tell whether it landed — and a naive retry appends the same media **again**, producing duplicate attachments on the post. With an `Idempotency-Key`, the retry replays the original response instead. **Always send one when the body contains `mediaUrls`.**

```javascript
const key = crypto.randomUUID();   // reuse this exact key for every retry

await fetch(`https://api.publora.com/api/v1/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': process.env.PUBLORA_API_KEY,
    'Idempotency-Key': key
  },
  body: JSON.stringify({
    mediaUrls: ['https://example.com/photo.jpg']
  })
});
```

Generate the key **once per logical update** and reuse it across retries. Generating a fresh key per attempt defeats the protection entirely.

## Response

```json
{
  "success": true,
  "message": "Post updated successfully",
  "scheduledTime": "2026-03-15T10:00:00.000Z",
  "postGroup": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "scheduled",
    "scheduledTime": "2026-03-15T10:00:00.000Z"
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `scheduledTime` | string/null | **Always present.** The effective stored time after any permitted past-time clamp. `null` if the post has no scheduled time. |
| `postGroup.scheduledTime` | string | The same value; included only when the post has a scheduled time set. |
| `warnings` | array | Present only when something was adjusted — e.g. a `SCHEDULED_TIME_COERCED` entry. See [Past scheduled times](#past-scheduled-times). |
| `mediaValidationStatus` | string | Present as `"pending"` only when media validation had not completed by the time the response was sent. |

## Examples

### Reschedule a post

#### JavaScript (fetch)

```javascript
const response = await fetch(
  `https://api.publora.com/api/v1/update-post/${postGroupId}`,
  {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      scheduledTime: '2026-03-15T10:00:00.000Z'
    })
  }
);
```

#### Python (requests)

```python
response = requests.put(
    f'https://api.publora.com/api/v1/update-post/{post_group_id}',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={'scheduledTime': '2026-03-15T10:00:00.000Z'}
)
```

#### cURL

```bash
curl -X PUT https://api.publora.com/api/v1/update-post/507f1f77bcf86cd799439011 \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{"scheduledTime": "2026-03-15T10:00:00.000Z"}'
```

### Change a draft to scheduled

```javascript
await fetch(`https://api.publora.com/api/v1/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: '2026-04-01T12:00:00.000Z'
  })
});
```

### Pause a scheduled post (move to draft)

```python
response = requests.put(
    f'https://api.publora.com/api/v1/update-post/{post_group_id}',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={'status': 'draft'}
)
```

### Node.js (axios)

```javascript
const axios = require('axios');

const client = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
});

async function reschedulePost(postGroupId, newTime) {
  const { data } = await client.put(`/update-post/${postGroupId}`, {
    scheduledTime: newTime
  });
  return data;
}

// Usage
await reschedulePost('507f1f77bcf86cd799439011', '2026-03-20T14:00:00.000Z');
```

### With Error Handling

```javascript
async function updatePostSafely(postGroupId, updates) {
  try {
    const response = await fetch(
      `https://api.publora.com/api/v1/update-post/${postGroupId}`,
      {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json',
          'x-publora-key': process.env.PUBLORA_API_KEY
        },
        body: JSON.stringify(updates)
      }
    );

    const data = await response.json();

    if (!response.ok) {
      switch (response.status) {
        case 400:
          if (data.error?.includes('status')) {
            throw new Error('Post is in a non-editable status. Cannot update.');
          }
          throw new Error(data.error || 'Invalid request');
        case 401:
          throw new Error('Authentication failed. Check your API key.');
        case 403:
          throw new Error('API access is not enabled for this account.');
        case 404:
          throw new Error('Post not found. It may have been deleted.');
        default:
          throw new Error(data.error || `HTTP ${response.status}`);
      }
    }

    return data;
  } catch (error) {
    console.error('Failed to update post:', error.message);
    throw error;
  }
}
```

```python
def update_post_safely(post_group_id, updates):
    """Update a post with comprehensive error handling."""
    try:
        response = requests.put(
            f'https://api.publora.com/api/v1/update-post/{post_group_id}',
            headers={
                'Content-Type': 'application/json',
                'x-publora-key': os.environ['PUBLORA_API_KEY']
            },
            json=updates
        )

        data = response.json()

        if response.status_code == 400:
            error = data.get('error', '')
            if 'status' in error.lower():
                raise ValueError('Post is in a non-editable status. Cannot update.')
            raise ValueError(error or 'Invalid request')

        if response.status_code == 401:
            raise ValueError('Authentication failed. Check your API key.')

        if response.status_code == 403:
            raise ValueError('API access is not enabled for this account.')

        if response.status_code == 404:
            raise ValueError('Post not found. It may have been deleted.')

        response.raise_for_status()
        return data

    except requests.RequestException as e:
        print(f'Failed to update post: {e}')
        raise
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"At least one of status, scheduledTime, platformSettings, or mediaUrls must be provided"` | None of the four fields was provided (see note below) |
| 400 | `"Unknown platformSettings path: {path}"` — code `PLATFORM_SETTING_UNKNOWN` | An unrecognized top-level platform or nested settings key. The response includes `field` with the exact dotted path. Nothing is persisted. |
| 400 | `"platformSettings.linkedin.*"` validation errors | Invalid LinkedIn repost settings (bad URN shape, bad visibility, `repostEnabled`/`repostParentUrn` mismatch) — see [Create Post → LinkedIn Repost Settings](./create-post.md#linkedin-repost-settings) |
| 400 | `"LinkedIn company-page reposts cannot use CONNECTIONS visibility; choose PUBLIC"` | `repostVisibility: "CONNECTIONS"` on a scheduled group that targets a LinkedIn company-page connection |
| 400 | `"Scheduled time is in the past. Server time is {t} UTC."` — code `SCHEDULED_TIME_IN_PAST` | The requested or stored time is 5+ minutes in the past while strict mode is active (scheduled for **2026-08-25** unless configuration overrides it). Includes `serverTime`. |
| 400 | `"Idempotency-Key request body is too deeply nested"` — code `IDEMPOTENCY_BODY_TOO_COMPLEX` | The request body sent with an `Idempotency-Key` exceeds the nesting limit and cannot be hashed |
| 409 | `"A request with this idempotency key is still in flight"` — code `IDEMPOTENCY_IN_FLIGHT` | Another request with the same `Idempotency-Key` is still being processed. Retry shortly. |
| 422 | `"Idempotency key was already used with a different request body"` — code `IDEMPOTENCY_KEY_CONFLICT` | The same `Idempotency-Key` was reused with a different body. Use a new key. |
| 400 | `"Status must be either 'draft' or 'scheduled'"` | Invalid status value |
| 400 | `"Invalid scheduled time format"` | Malformed datetime string |
| 400 | `"Cannot update post: post is currently in {status} status"` | Post is in any status other than `draft` or `scheduled` |
| 400 | `"Invalid x-publora-user-id"` | The `x-publora-user-id` header value is not a valid ObjectId format |
| 401 | `"API key is required"` | Missing `x-publora-key` header |
| 401 | `"Invalid API key"` | `x-publora-key` value is incorrect or revoked |
| 401 | `"Invalid API key owner"` | The API key exists but its owner account could not be found |
| 403 | `"API access is not enabled for this account"` | No active subscription or API access not enabled |
| 403 | `"MCP access is not enabled for this account"` | The account does not have MCP access enabled (MCP-only keys) |
| 403 | `"Workspace access is not enabled for this key"` | The API key does not have workspace/managed-user permissions |
| 403 | `"User is not managed by key"` | The `x-publora-user-id` references a user not managed by this API key |
| 403 | `LimitExceededError` (structured JSON) | Rescheduling would exceed the account's posting limits for the target time slot (see below) |
| 404 | `"Post group not found"` | Invalid ID or post belongs to another user |
| 500 | `"Failed to update post"` | Malformed post group ID or internal server error |
| 500 | `"Internal server error"` | Unexpected server error in middleware |

> **Note:** The `status` field must be a **non-empty string**. The server uses a JavaScript falsy check, so empty strings (`""`), `null`, and `0` are all treated as absent. If `status` and `scheduledTime` are both falsy and neither `platformSettings` nor `mediaUrls` is present, you will receive the "At least one of status, scheduledTime, platformSettings, or mediaUrls must be provided" error.

> **Note:** If `x-publora-user-id` matches the API key owner, no workspace check is triggered — the header is effectively a no-op in that case.

> **Note:** If the `postGroupId` is not a valid MongoDB ObjectId format (e.g., too short, contains invalid characters), the server returns a **500** error (`"Failed to update post"`) instead of a **400** validation error. Ensure you pass only valid ObjectId strings received from the create-post or list-posts endpoints.

### Limit Exceeded Error Format

When a **403** limit error is returned, the response body is a structured JSON object (not a simple string):

```json
{
  "error": "Post limit reached",
  "code": "POST_LIMIT_REACHED",
  "metric": "posts.platform_monthly",
  "message": "Monthly post limit reached. Your Pro plan allows 50 platform posts per month.",
  "limit": 50,
  "used": 48,
  "requested": 3,
  "remaining": 2,
  "periodStart": "2026-03-01T00:00:00.000Z",
  "periodEnd": "2026-04-01T00:00:00.000Z",
  "planName": "Pro"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `error` | string | Descriptive error string (see values below) |
| `code` | string | Error code identifying which limit was hit (see values below) |
| `metric` | string | Which limit metric was exceeded (see values below) |
| `message` | string | Human-readable description of the limit violation |
| `limit` | number | Maximum allowed value for this metric |
| `used` | number/null | How many have been used in the current period (`null` for date-based limits) |
| `requested` | number/null | How many were requested in this operation (`null` for date-based limits) |
| `remaining` | number/null | How many are still available (`null` for date-based limits) |
| `periodStart` | string | ISO 8601 start of the current billing/limit period |
| `periodEnd` | string | ISO 8601 end of the current billing/limit period |
| `planName` | string | The user's current plan name |

**Error codes and their corresponding `error` and `metric` values:**

| `code` | `error` | `metric` |
|--------|---------|----------|
| `POST_LIMIT_REACHED` | `"Post limit reached"` | `"posts.platform_monthly"` |
| `SCHEDULED_POST_LIMIT_REACHED` | `"Scheduled post limit reached"` | `"posts.scheduled_active"` |
| `SCHEDULE_HORIZON_REACHED` | `"Schedule horizon reached"` | `"posts.schedule_horizon_days"` |
| `PLATFORM_NOT_AVAILABLE` | `"Platform not available"` | `"posts.platform_monthly"` |
| `CONNECTIONS_OVER_LIMIT` | `"Account over channel limit"` | `"connections.total"` |

> **Note:** For `SCHEDULE_HORIZON_REACHED`, the `used`, `requested`, and `remaining` fields are `null` since this limit is date-based rather than count-based.

> **Note:** Additional top-level context depends on the code: `scheduledTime`/`maxScheduledDate` for the schedule horizon; `limitScope` and (for connection scope) `blockedPlatforms` for monthly posts; workspace fields for the scheduled queue; and platform allowlist fields for availability checks.

> **Note (low priority):** When a `LimitExceededError` is triggered, the API also sends a limit-reached notification email to the account owner as a side effect. This is an internal behavior and does not affect the API response.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
