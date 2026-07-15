# How to Update a Scheduled Post Before Publication

Update the timing or status of an already-scheduled post using the Publora API. This guide covers the complete workflow with error handling and edge cases.

## Overview

You can modify a scheduled post before it publishes by:
- **Rescheduling** to a different time
- **Pausing** (converting to draft)
- **Resuming** a paused draft

**Important:** You can only update posts with status `draft` or `scheduled`. Posts that are `published`, `failed`, `pending`, or `processing` cannot be modified.

## Prerequisites

1. **API Key**: Get your key from the [Publora Dashboard](https://publora.com/dashboard)
2. **Post Group ID**: The ID returned when you created the post
3. **Post must be updatable**: Status must be `draft` or `scheduled`

## API Reference

**Endpoint:**
```
PUT https://api.publora.com/api/v1/update-post/:postGroupId
```

**Headers:**
```
Content-Type: application/json
x-publora-key: YOUR_API_KEY
```

**Request Body** (at least one field required):
```json
{
  "status": "scheduled",
  "scheduledTime": "2026-03-15T14:00:00.000Z"
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `status` | string | No* | `"draft"` or `"scheduled"` |
| `scheduledTime` | string | No* | ISO 8601 UTC datetime (must be future) |

*At least one of `status` or `scheduledTime` must be provided.

**Success Response (200):**
```json
{
  "success": true,
  "message": "Post updated successfully",
  "postGroup": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "scheduled",
    "scheduledTime": "2026-03-15T14:00:00.000Z"
  }
}
```

## Complete JavaScript Implementation

```javascript
const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY;
const BASE_URL = 'https://api.publora.com/api/v1';

/**
 * Update a scheduled post before publication
 * @param {string} postGroupId - The post group ID to update
 * @param {Object} options - Update options
 * @param {string} [options.scheduledTime] - New ISO 8601 UTC datetime
 * @param {string} [options.status] - New status: 'draft' or 'scheduled'
 * @returns {Promise<Object>} Updated post data
 */
async function updateScheduledPost(postGroupId, options = {}) {
  // Validate inputs
  if (!postGroupId) {
    throw new Error('postGroupId is required');
  }

  if (!options.scheduledTime && !options.status) {
    throw new Error('Either scheduledTime or status must be provided');
  }

  // Validate scheduledTime is in the future
  if (options.scheduledTime) {
    const scheduledDate = new Date(options.scheduledTime);
    if (isNaN(scheduledDate.getTime())) {
      throw new Error('Invalid scheduledTime format. Use ISO 8601: YYYY-MM-DDTHH:mm:ss.sssZ');
    }
    if (scheduledDate <= new Date()) {
      throw new Error('scheduledTime must be in the future');
    }
  }

  // Validate status
  if (options.status && !['draft', 'scheduled'].includes(options.status)) {
    throw new Error('status must be "draft" or "scheduled"');
  }

  // Build request body
  const body = {};
  if (options.scheduledTime) body.scheduledTime = options.scheduledTime;
  if (options.status) body.status = options.status;

  // Make API request
  const response = await fetch(`${BASE_URL}/update-post/${postGroupId}`, {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify(body)
  });

  const data = await response.json();

  // Handle errors
  if (!response.ok) {
    const error = new Error(data.error || `HTTP ${response.status}`);
    error.status = response.status;
    error.response = data;

    switch (response.status) {
      case 400:
        if (data.code === 'SCHEDULED_TIME_IN_PAST') {
          error.message = 'Cannot schedule in the past. Use a future datetime.';
        } else if (data.error?.includes('status')) {
          error.message = 'Post cannot be updated. It may be published, failed, or processing.';
        }
        break;
      case 401:
        error.message = 'Invalid API key. Check your x-publora-key header.';
        break;
      case 404:
        error.message = 'Post not found. It may have been deleted or belongs to another user.';
        break;
    }

    throw error;
  }

  return data;
}

// ============ USAGE EXAMPLES ============

// Example 1: Reschedule to a new time
async function reschedulePost(postGroupId, newDateTime) {
  try {
    const result = await updateScheduledPost(postGroupId, {
      scheduledTime: newDateTime
    });
    console.log('Post rescheduled:', result.postGroup.scheduledTime);
    return result;
  } catch (error) {
    console.error('Failed to reschedule:', error.message);
    throw error;
  }
}

// Example 2: Pause a scheduled post (move to draft)
async function pausePost(postGroupId) {
  try {
    const result = await updateScheduledPost(postGroupId, {
      status: 'draft'
    });
    console.log('Post paused (moved to draft)');
    return result;
  } catch (error) {
    console.error('Failed to pause:', error.message);
    throw error;
  }
}

// Example 3: Resume a draft with new schedule
async function resumePost(postGroupId, scheduledTime) {
  try {
    const result = await updateScheduledPost(postGroupId, {
      status: 'scheduled',
      scheduledTime: scheduledTime
    });
    console.log('Post scheduled for:', result.postGroup.scheduledTime);
    return result;
  } catch (error) {
    console.error('Failed to resume:', error.message);
    throw error;
  }
}

// Example 4: Schedule for tomorrow at 9am UTC
function getTomorrowAt9AM() {
  const tomorrow = new Date();
  tomorrow.setDate(tomorrow.getDate() + 1);
  tomorrow.setUTCHours(9, 0, 0, 0);
  return tomorrow.toISOString();
}

// Run examples
const postGroupId = '507f1f77bcf86cd799439011';

await reschedulePost(postGroupId, '2026-03-20T14:00:00.000Z');
await pausePost(postGroupId);
await resumePost(postGroupId, getTomorrowAt9AM());
```

## Complete Python Implementation

```python
import os
import requests
from datetime import datetime, timezone, timedelta
from typing import Optional

PUBLORA_API_KEY = os.environ.get('PUBLORA_API_KEY')
BASE_URL = 'https://api.publora.com/api/v1'


class PubloraError(Exception):
    """Custom exception for Publora API errors."""
    def __init__(self, message: str, status_code: int = None, response: dict = None):
        super().__init__(message)
        self.status_code = status_code
        self.response = response


def update_scheduled_post(
    post_group_id: str,
    scheduled_time: Optional[str] = None,
    status: Optional[str] = None
) -> dict:
    """
    Update a scheduled post before publication.

    Args:
        post_group_id: The post group ID to update
        scheduled_time: New ISO 8601 UTC datetime (e.g., '2026-03-15T14:00:00.000Z')
        status: New status - 'draft' or 'scheduled'

    Returns:
        dict: Updated post data with success status

    Raises:
        PubloraError: If the API request fails
        ValueError: If inputs are invalid
    """
    # Validate inputs
    if not post_group_id:
        raise ValueError('post_group_id is required')

    if not scheduled_time and not status:
        raise ValueError('Either scheduled_time or status must be provided')

    # Validate scheduled_time is in the future
    if scheduled_time:
        try:
            scheduled_dt = datetime.fromisoformat(scheduled_time.replace('Z', '+00:00'))
            if scheduled_dt <= datetime.now(timezone.utc):
                raise ValueError('scheduled_time must be in the future')
        except ValueError as e:
            if 'future' in str(e):
                raise
            raise ValueError('Invalid scheduled_time format. Use ISO 8601: YYYY-MM-DDTHH:mm:ss.sssZ')

    # Validate status
    if status and status not in ['draft', 'scheduled']:
        raise ValueError('status must be "draft" or "scheduled"')

    # Build request body
    body = {}
    if scheduled_time:
        body['scheduledTime'] = scheduled_time
    if status:
        body['status'] = status

    # Make API request
    response = requests.put(
        f'{BASE_URL}/update-post/{post_group_id}',
        headers={
            'Content-Type': 'application/json',
            'x-publora-key': PUBLORA_API_KEY
        },
        json=body
    )

    data = response.json()

    # Handle errors
    if not response.ok:
        error_message = data.get('error', f'HTTP {response.status_code}')

        if response.status_code == 400:
            if data.get('code') == 'SCHEDULED_TIME_IN_PAST':
                error_message = 'Cannot schedule in the past. Use a future datetime.'
            elif 'status' in error_message.lower():
                error_message = 'Post cannot be updated. It may be published, failed, or processing.'
        elif response.status_code == 401:
            error_message = 'Invalid API key. Check your x-publora-key header.'
        elif response.status_code == 404:
            error_message = 'Post not found. It may have been deleted or belongs to another user.'

        raise PubloraError(error_message, response.status_code, data)

    return data


# ============ USAGE EXAMPLES ============

def reschedule_post(post_group_id: str, new_datetime: str) -> dict:
    """Reschedule a post to a new time."""
    try:
        result = update_scheduled_post(post_group_id, scheduled_time=new_datetime)
        print(f"Post rescheduled: {result['postGroup']['scheduledTime']}")
        return result
    except (PubloraError, ValueError) as e:
        print(f"Failed to reschedule: {e}")
        raise


def pause_post(post_group_id: str) -> dict:
    """Pause a scheduled post by moving it to draft."""
    try:
        result = update_scheduled_post(post_group_id, status='draft')
        print("Post paused (moved to draft)")
        return result
    except (PubloraError, ValueError) as e:
        print(f"Failed to pause: {e}")
        raise


def resume_post(post_group_id: str, scheduled_time: str) -> dict:
    """Resume a draft post with a new schedule."""
    try:
        result = update_scheduled_post(
            post_group_id,
            status='scheduled',
            scheduled_time=scheduled_time
        )
        print(f"Post scheduled for: {result['postGroup']['scheduledTime']}")
        return result
    except (PubloraError, ValueError) as e:
        print(f"Failed to resume: {e}")
        raise


def get_tomorrow_at_9am() -> str:
    """Get ISO 8601 string for tomorrow at 9am UTC."""
    tomorrow = datetime.now(timezone.utc) + timedelta(days=1)
    scheduled = tomorrow.replace(hour=9, minute=0, second=0, microsecond=0)
    return scheduled.strftime('%Y-%m-%dT%H:%M:%S.000Z')


# Run examples
if __name__ == '__main__':
    post_group_id = '507f1f77bcf86cd799439011'

    reschedule_post(post_group_id, '2026-03-20T14:00:00.000Z')
    pause_post(post_group_id)
    resume_post(post_group_id, get_tomorrow_at_9am())
```

## cURL Examples

### Reschedule a post

```bash
curl -X PUT "https://api.publora.com/api/v1/update-post/507f1f77bcf86cd799439011" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{"scheduledTime": "2026-03-15T14:00:00.000Z"}'
```

### Pause a post (move to draft)

```bash
curl -X PUT "https://api.publora.com/api/v1/update-post/507f1f77bcf86cd799439011" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{"status": "draft"}'
```

### Resume with new schedule

```bash
curl -X PUT "https://api.publora.com/api/v1/update-post/507f1f77bcf86cd799439011" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{"status": "scheduled", "scheduledTime": "2026-03-15T14:00:00.000Z"}'
```

## Edge Cases and Error Handling

### Posts That Cannot Be Updated

| Status | Can Update? | Reason |
|--------|-------------|--------|
| `draft` | Yes | Not yet scheduled |
| `scheduled` | Yes | Waiting for publication time |
| `pending` | No | Being processed by scheduler |
| `processing` | No | Currently publishing |
| `published` | No | Already live on platform |
| `failed` | No | Publication failed |

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `"Scheduled time is in the past. Server time is <ISO> UTC."` (`code: "SCHEDULED_TIME_IN_PAST"`) | DateTime is 5+ minutes before server time, and the stricter behaviour is active (from **2026-08-25**) | Use a future datetime; compare the response's `serverTime` to your clock. Under 5 min late is clamped to now with a `SCHEDULED_TIME_COERCED` warning instead. See [Scheduling](./scheduling.md#past-scheduled-times) |
| `"Cannot update post: post is currently in published status"` | Post already published | Cannot modify published posts |
| `"Post group not found"` | Invalid ID or wrong user | Verify postGroupId and API key |
| `"Invalid API key"` | Bad x-publora-key header | Check API key in dashboard |
| `"Invalid scheduled time format"` | Malformed datetime | Use ISO 8601: `YYYY-MM-DDTHH:mm:ss.sssZ` |

> **Heads-up:** a successful update can still carry `warnings` — `[{ code: "SCHEDULED_TIME_COERCED", requested, effective }]` means your time was in the past and the post was moved to `effective` (server time). Don't discard the warnings array; re-read the post with `GET /get-post` to see the stored `scheduledTime`.

### Checking Post Status Before Update

```javascript
async function canUpdatePost(postGroupId) {
  const response = await fetch(
    `https://api.publora.com/api/v1/get-post/${postGroupId}`,
    { headers: { 'x-publora-key': PUBLORA_API_KEY } }
  );

  const data = await response.json();
  const status = data.posts?.[0]?.status;

  return ['draft', 'scheduled'].includes(status);
}

// Usage
if (await canUpdatePost(postGroupId)) {
  await updateScheduledPost(postGroupId, { scheduledTime: newTime });
} else {
  console.log('Post cannot be updated - already published or processing');
}
```

## Complete Workflow: Create Draft and Schedule Later

```javascript
// Step 1: Create a draft post
const createResponse = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'My post content here',
    platforms: ['linkedin-ABC123', 'twitter-XYZ789']
    // No scheduledTime = creates as draft
  })
});

const { postGroupId } = await createResponse.json();
console.log('Draft created:', postGroupId);

// Step 2: Later, schedule the draft
const updateResponse = await fetch(
  `https://api.publora.com/api/v1/update-post/${postGroupId}`,
  {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      status: 'scheduled',
      scheduledTime: '2026-03-15T14:00:00.000Z'
    })
  }
);

const result = await updateResponse.json();
console.log('Post scheduled for:', result.postGroup.scheduledTime);
```

## Related Guides

- [Create Post](/docs/endpoints/create-post.md) - Create new posts
- [Get Post](/docs/endpoints/get-post.md) - Check post status
- [Delete Post](/docs/endpoints/delete-post.md) - Remove posts
- [Scheduling Guide](/docs/guides/scheduling.md) - Scheduling best practices
