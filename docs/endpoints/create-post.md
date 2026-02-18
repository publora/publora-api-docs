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
| `Content-Type` | Yes | `application/json` |

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | Post text content |
| `platforms` | string[] | Yes | Array of platform connection IDs (format: `platform-platformId`) |
| `scheduledTime` | string | No | ISO 8601 UTC datetime. If omitted, queued for next available slot |

## Response

```json
{
  "success": true,
  "postGroupId": "507f1f77bcf86cd799439011"
}
```

Use the `postGroupId` to track, update, or delete the post.

## Default Platform Settings

When creating via the API, these defaults are applied automatically:

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
    "title": ""
  }
}
```

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
| 400 | `"Content is required"` | Missing `content` field |
| 400 | `"Platforms are required"` | Missing or empty `platforms` array |
| 400 | `"Invalid scheduled time"` | `scheduledTime` is in the past or malformed |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 403 | `"Subscription required"` | No active subscription |
| 403 | `"Free plan limit reached"` | Free tier: max 5 pending posts |
| 500 | `"Internal server error"` | Unexpected server error |

## Post Statuses

After creation, the post goes through these states:

```
draft → scheduled → processing → published
                                → failed
                                → partially_published
```

- **draft**: Saved but not scheduled
- **scheduled**: Will be published at `scheduledTime`
- **processing**: Currently being sent to platforms
- **published**: Successfully posted on all platforms
- **failed**: Failed on all platforms
- **partially_published**: Succeeded on some, failed on others


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
