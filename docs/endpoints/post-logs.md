# Post Logs

Get detailed publish attempt history for a post group. Useful for debugging failed posts and understanding the publish timeline.

## Endpoint

```
GET https://api.publora.com/api/v1/post-logs/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Set to `mcp` for MCP tool access |

> **MCP access:** If the `x-publora-client: mcp` header is sent, the server validates `entitlements.features.mcpAccess` and returns `403 "MCP access is not enabled for this account"` if the feature is not enabled.

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID |

> **Note:** Results are limited to the first 100 log entries per post group (sorted ascending by creation time, oldest first).

## Response

```json
{
  "success": true,
  "logs": [
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
      "postGroupId": "507f1f77bcf86cd799439011",
      "postId": "507f1f77bcf86cd799439012",
      "platform": "linkedin",
      "event": "processing",
      "createdAt": "2026-02-22T14:30:00.000Z"
    },
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d1",
      "postGroupId": "507f1f77bcf86cd799439011",
      "postId": "507f1f77bcf86cd799439012",
      "platform": "linkedin",
      "event": "publish_attempted",
      "retryCount": 0,
      "createdAt": "2026-02-22T14:30:01.000Z"
    },
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d2",
      "postGroupId": "507f1f77bcf86cd799439011",
      "postId": "507f1f77bcf86cd799439012",
      "platform": "linkedin",
      "event": "publish_succeeded",
      "createdAt": "2026-02-22T14:30:03.000Z"
    },
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d3",
      "postGroupId": "507f1f77bcf86cd799439011",
      "event": "processing",
      "createdAt": "2026-02-22T14:30:00.000Z"
    },
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d4",
      "postGroupId": "507f1f77bcf86cd799439011",
      "postId": "507f1f77bcf86cd799439013",
      "platform": "threads",
      "event": "publish_attempted",
      "retryCount": 1,
      "createdAt": "2026-02-22T14:30:01.000Z"
    },
    {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d5",
      "postGroupId": "507f1f77bcf86cd799439011",
      "postId": "507f1f77bcf86cd799439013",
      "platform": "threads",
      "event": "publish_failed",
      "error": "Error validating access token",
      "httpStatus": 401,
      "retryCount": 1,
      "metadata": {
        "errorCode": "PLATFORM_AUTH_EXPIRED",
        "retryable": false
      },
      "createdAt": "2026-02-22T14:30:02.000Z"
    }
  ]
}
```

> **Note:** Response also includes `updatedAt` (auto-managed by Mongoose) and `__v` (Mongoose version key) fields, omitted from examples for brevity.

## Log Fields

| Field | Type | Description |
|-------|------|-------------|
| `_id` | string | Unique log entry ID |
| `postGroupId` | string | Parent post group ID |
| `postId` | string/undefined | Individual platform post ID. May be absent for group-level events (e.g., initial `processing`). |
| `platform` | string/undefined | Platform name (twitter, linkedin, etc.). May be absent for group-level events. |
| `event` | string | Event type (see table below) |
| `error` | string or absent | Error message (for failed events). Absent from the response when there is no error (not `null`). |
| `httpStatus` | number/null/absent | HTTP status code from the platform. Scheduler-created logs may store it explicitly as `null`; other logs may omit it when not applicable. |
| `retryCount` | number or absent | Number of retry attempts. Absent if no retries have occurred -- do not assume it defaults to 0. |
| `metadata` | object or absent | Additional event data. Absent from the response when there is no metadata (not `null`). |
| `createdAt` | string | Event timestamp |
| `updatedAt` | string | Last update timestamp (auto-managed by Mongoose) |
| `__v` | number | Mongoose version key. Present in responses but not part of the documented schema — can be safely ignored. |

## Event Types

| Event | Description |
|-------|-------------|
| `scheduled` | Post was scheduled |
| `processing` | Post started processing |
| `publish_attempted` | Publish request sent to platform |
| `publish_succeeded` | Successfully published |
| `publish_failed` | Publishing failed |
| `publish_retry` | A publish retry was recorded |
| `status_changed` | Post status changed |
| `media_finalize_succeeded` | Attached media finalization succeeded |
| `media_finalize_failed` | Attached media finalization failed |
| `linkedin_document_upload_succeeded` | LinkedIn PDF/document upload succeeded |
| `linkedin_document_upload_failed` | LinkedIn PDF/document upload failed |

## Examples

### JavaScript (fetch)

```javascript
const postGroupId = '507f1f77bcf86cd799439011';
const response = await fetch(
  `https://api.publora.com/api/v1/post-logs/${postGroupId}`,
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
const { logs } = await response.json();

// Group logs by platform
const byPlatform = logs.reduce((acc, log) => {
  if (!acc[log.platform]) acc[log.platform] = [];
  acc[log.platform].push(log);
  return acc;
}, {});

// Check for failures
const failures = logs.filter(l => l.event === 'publish_failed');
if (failures.length > 0) {
  console.log('Failed platforms:');
  failures.forEach(f => {
    console.log(`  ${f.platform}: ${f.error} (HTTP ${f.httpStatus})`);
  });
}
```

### Python (requests)

```python
import requests

post_group_id = '507f1f77bcf86cd799439011'
response = requests.get(
    f'https://api.publora.com/api/v1/post-logs/{post_group_id}',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
logs = response.json()['logs']

# Find failed publish attempts
failures = [l for l in logs if l['event'] == 'publish_failed']
for failure in failures:
    print(f"❌ {failure['platform']}: {failure['error']}")
    if failure.get('metadata', {}).get('retryable'):
        print("   (This error may succeed on retry)")
```

### cURL

```bash
curl https://api.publora.com/api/v1/post-logs/507f1f77bcf86cd799439011 \
  -H "x-publora-key: YOUR_API_KEY"
```

### Debug Failed Post

```javascript
async function debugFailedPost(postGroupId) {
  const [postResponse, logsResponse] = await Promise.all([
    fetch(`https://api.publora.com/api/v1/get-post/${postGroupId}`, {
      headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
    }),
    fetch(`https://api.publora.com/api/v1/post-logs/${postGroupId}`, {
      headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
    })
  ]);

  const postData = await postResponse.json();
  const logsData = await logsResponse.json();

  console.log('=== Post Status ===');
  for (const post of postData.posts) {
    console.log(`${post.platform}: ${post.status}`);
    if (post.error) {
      console.log(`  Error: ${post.error.code} - ${post.error.message}`);
      console.log(`  Retryable: ${post.error.retryable}`);
    }
  }

  console.log('\n=== Timeline ===');
  for (const log of logsData.logs) {
    const time = new Date(log.createdAt).toISOString();
    console.log(`[${time}] ${log.platform}: ${log.event}`);
    if (log.error) {
      console.log(`  └─ ${log.error}`);
    }
  }
}
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Invalid x-publora-user-id"` | The `x-publora-user-id` header value is not a valid ID |
| 401 | `"API key is required"` | `x-publora-key` header is missing entirely |
| 401 | `"Invalid API key"` | `x-publora-key` is present but invalid |
| 401 | `"Invalid API key owner"` | API key exists but the associated user account was not found |
| 403 | `"API access is not enabled for this account"` | The user's account does not have API access enabled |
| 403 | `"MCP access is not enabled for this account"` | The `x-publora-client: mcp` header was sent but `entitlements.features.mcpAccess` is not enabled |
| 403 | `"Workspace access is not enabled for this key"` | API key does not have workspace access enabled |
| 403 | `"User is not managed by key"` | Managed user does not belong to the API key owner's workspace |
| 404 | `"Post group not found"` | Invalid ID or post belongs to another user |
| 500 | `"Failed to get post logs"` | Server error |

> **Note:** If an invalid ObjectId format is passed as `postGroupId` (e.g., a non-hex string or wrong length), the endpoint returns `500 "Failed to get post logs"` instead of `404 "Post group not found"`. This is because Mongoose throws a `CastError` for invalid ObjectIds, which is caught by the generic error handler before the post group lookup can return a 404.

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
