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

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID to update |

## Request Body

At least one of `status`, `scheduledTime`, or `platformSettings` must be provided. Missing all three returns `400 { "error": "At least one of status, scheduledTime, or platformSettings must be provided" }`.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | `"draft"` or `"scheduled"` |
| `scheduledTime` | string | No | New ISO 8601 UTC datetime |
| `platformSettings` | object | No | Per-platform settings (same shape and allowlist as in [create-post](./create-post.md#platformsettings)). Merged with existing settings server-side — fields you omit are preserved. Only `tiktok`, `instagram`, `youtube`, `threads`, and `telegram` keys are accepted; other platform keys are silently dropped. |

> **YouTube playlist & thumbnail (partial update):** `platformSettings` supports partial per-platform updates, so you can change `youtube.playlist` or set/clear `youtube.thumbnail` without resending the other YouTube fields. The **thumbnail can only be set here** (not on `create-post`, which has no `postGroupId` yet). Clear either by sending empty strings — `youtube.playlist` as `{ "id": "", "platformId": "" }`, or `youtube.thumbnail` as `{ "url": "" }`. Playlist or thumbnail changes return **`409`** while the post is publishing/processing (e.g. *"Post group is currently being published; cannot change YouTube playlist. Retry once publishing completes."*). See [YouTube → Platform-Specific Settings](../platforms/youtube.md#platform-specific-settings).

> **Instagram Reels cover:** set or change `instagram.coverUrl` (alias: `cover_url`) here — a JPEG image used as the Reel's cover. Either pass a publicly accessible http(s) URL, or [upload a cover file](upload-instagram-cover.md) (JPEG/PNG/WebP up to 8 MB) and set the returned URL. Send `{ "instagram": { "coverUrl": "" } }` to clear it (the cover falls back to automatic/frame-based selection). Non-JPEG or non-http(s) URLs are rejected with `400`; a change is rejected with `409` while the post is publishing. See [Instagram → Platform-Specific Settings](../platforms/instagram.md#platform-specific-settings).

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

- **Past times:** If the scheduled time is in the past, it is silently set to the current time
- **No scheduled time:** If the resulting status is `scheduled` and neither the request body nor the existing post has a `scheduledTime`, it defaults to the current time (`new Date()`). This default does not apply when updating to `draft` — the existing value is preserved
- **Maximum:** Recommended within 2 months for best reliability
- **Timezone:** Always use UTC (Z suffix or +00:00 offset)

### Scheduling Limits

When rescheduling a post (changing `scheduledTime`), the update flow performs two validations:

1. **Platform availability check** (`assertPlatformsAllowed`): Verifies that all platforms targeted by the post are still available on the user's current plan. If a platform is no longer allowed, a `PLATFORM_NOT_AVAILABLE` error is returned.
2. **Limits check**: Validates that the new time does not exceed the account's posting limits for the target time slot.

If either check fails, the request returns a **403** error with a `LimitExceededError` message describing which limit was hit.

> **Note:** The limits service may automatically adjust the `scheduledTime` to comply with minimum interval constraints between posts. If this happens, the limits service `scheduledTime` takes priority over the user-provided time. The response will contain the adjusted time in the `postGroup.scheduledTime` field — always use the returned value rather than assuming the requested time was used as-is.

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

## Response

```json
{
  "success": true,
  "message": "Post updated successfully",
  "postGroup": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "scheduled",
    "scheduledTime": "2026-03-15T10:00:00.000Z"
  }
}
```

**Note:** The `scheduledTime` field is conditionally included in the response only if the post has a scheduled time set.

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
| 400 | `"Either status or scheduledTime must be provided"` | Neither field was provided in the request (see note below) |
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

> **Note:** The `status` field must be a **non-empty string**. The server uses a JavaScript falsy check, so empty strings (`""`), `null`, and `0` are all treated as absent. If both `status` and `scheduledTime` are falsy, you will receive the "Either status or scheduledTime must be provided" error.

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

> **Note:** The response may include additional context fields depending on the error code. These can include `scheduledTime`, `scope`, `blockedPlatforms`, `channelBreakdown`, `disallowedPlatforms`, and `allowedPlatforms`. These fields are spread from the error context and provide extra details about why the limit was exceeded.

> **Note (low priority):** When a `LimitExceededError` is triggered, the API also sends a limit-reached notification email to the account owner as a side effect. This is an internal behavior and does not affect the API response.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
