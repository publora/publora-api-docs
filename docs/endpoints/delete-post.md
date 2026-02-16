# Delete Post

Delete a scheduled or draft post. Also removes associated media files from storage.

## Endpoint

```
DELETE https://api.publora.com/api/v1/delete-post/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID to delete |

## Response

```json
{
  "success": true
}
```

## Examples

### JavaScript (fetch)

```javascript
const response = await fetch(
  `https://api.publora.com/api/v1/delete-post/${postGroupId}`,
  {
    method: 'DELETE',
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);
const data = await response.json();
console.log(data.success); // true
```

### Python (requests)

```python
response = requests.delete(
    f'https://api.publora.com/api/v1/delete-post/{post_group_id}',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
print(response.json()['success'])  # True
```

### cURL

```bash
curl -X DELETE https://api.publora.com/api/v1/delete-post/507f1f77bcf86cd799439011 \
  -H "x-publora-key: YOUR_API_KEY"
```

### Node.js (axios)

```javascript
const axios = require('axios');

await axios.delete(
  `https://api.publora.com/api/v1/delete-post/${postGroupId}`,
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"Post group not found"` | Invalid ID or post belongs to another user |

## What Gets Deleted

- The post group record
- All individual platform post records
- All associated media files from S3 storage
- Media file database records
