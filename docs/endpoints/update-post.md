# Update Post

Modify the scheduling time or status of an existing post.

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

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | `"draft"` or `"scheduled"` |
| `scheduledTime` | string | No | New ISO 8601 UTC datetime |

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

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Status must be draft or scheduled"` | Invalid status value |
| 400 | `"Cannot update published or failed posts"` | Post already published/failed |
| 400 | `"Invalid scheduled time"` | Time is in the past or malformed |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"Post group not found"` | Invalid ID or post belongs to another user |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
