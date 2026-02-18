# Get Post

Retrieve details and publishing status of a post group.

## Endpoint

```
GET https://api.publora.com/api/v1/get-post/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID returned from create-post |

## Response

```json
{
  "success": true,
  "postGroupId": "507f1f77bcf86cd799439011",
  "posts": [
    {
      "platform": "twitter",
      "platformId": "123456789",
      "content": "Excited to share our new product launch! 🚀",
      "status": "published",
      "postedId": "1234567890123456789"
    },
    {
      "platform": "linkedin",
      "platformId": "ABC123",
      "content": "Excited to share our new product launch! 🚀",
      "status": "published",
      "postedId": "urn:li:share:7654321"
    }
  ]
}
```

## Post Status Values

| Status | Meaning |
|--------|---------|
| `draft` | Saved but not yet scheduled |
| `scheduled` | Waiting for scheduled time |
| `pending` | Being processed by scheduler |
| `processing` | Currently publishing to platform |
| `published` | Successfully posted |
| `failed` | Publishing failed |

## Examples

### JavaScript (fetch)

```javascript
const postGroupId = '507f1f77bcf86cd799439011';
const response = await fetch(
  `https://api.publora.com/api/v1/get-post/${postGroupId}`,
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
const data = await response.json();

for (const post of data.posts) {
  console.log(`${post.platform}: ${post.status}`);
  if (post.postedId) {
    console.log(`  Platform post ID: ${post.postedId}`);
  }
}
```

### Python (requests)

```python
import requests

post_group_id = '507f1f77bcf86cd799439011'
response = requests.get(
    f'https://api.publora.com/api/v1/get-post/{post_group_id}',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
data = response.json()

for post in data['posts']:
    print(f"{post['platform']}: {post['status']}")
```

### cURL

```bash
curl https://api.publora.com/api/v1/get-post/507f1f77bcf86cd799439011 \
  -H "x-publora-key: YOUR_API_KEY"
```

### Node.js (axios)

```javascript
const axios = require('axios');

const { data } = await axios.get(
  `https://api.publora.com/api/v1/get-post/${postGroupId}`,
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
// Check if all posts published successfully
const allPublished = data.posts.every(p => p.status === 'published');
console.log(`All published: ${allPublished}`);
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"Post group not found"` | Invalid ID or post belongs to another user |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
