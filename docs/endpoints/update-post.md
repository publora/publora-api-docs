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
| `Content-Type` | Yes | `application/json` |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID to update |

## Request Body

At least one of `status` or `scheduledTime` must be provided.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | `"draft"` or `"scheduled"` |
| `scheduledTime` | string | No | New ISO 8601 UTC datetime (must be in the future) |

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

- **Minimum:** Must be at least 1 minute in the future
- **Maximum:** No hard limit, but platform tokens may expire for far-future posts
- **Timezone:** Always use UTC (Z suffix). Local times will be rejected.

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
          if (data.error?.includes('past')) {
            throw new Error('Cannot schedule in the past. Use a future date.');
          }
          if (data.error?.includes('status')) {
            throw new Error('Post already published or failed. Cannot update.');
          }
          throw new Error(data.error || 'Invalid request');
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
            if 'past' in error.lower():
                raise ValueError('Cannot schedule in the past. Use a future date.')
            if 'status' in error.lower():
                raise ValueError('Post already published or failed. Cannot update.')
            raise ValueError(error or 'Invalid request')

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
| 400 | `"Either status or scheduledTime must be provided"` | Neither field was provided in the request |
| 400 | `"Status must be either 'draft' or 'scheduled'"` | Invalid status value |
| 400 | `"Invalid scheduled time format"` | Malformed datetime string |
| 400 | `"Scheduled time cannot be in the past"` | Provided time is before current time |
| 400 | `"Cannot update post: post is currently in {status} status"` | Post is in a status that cannot be updated (e.g., published, failed, pending, processing) |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"Post group not found"` | Invalid ID or post belongs to another user |
| 500 | `"Failed to update post"` | Server error during update operation |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
