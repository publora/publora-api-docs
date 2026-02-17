# Getting Started with Publora API

## Overview

Publora API lets you schedule and publish social media posts across 10 platforms from a single REST endpoint. Base URL: `https://api.publora.com`

## Step 1: Sign Up and Get an API Key

1. Create an account at [publora.com](https://publora.com)
2. Go to **Settings > API Keys**
3. Click **Generate API Key**
4. Copy the key immediately -- it's shown only once

Your key looks like: `sk_kzq5mjw.a1b2c3d4e5f6...`

## Step 2: Connect Social Accounts

Connect your social media accounts via the Publora dashboard. Each platform has its own OAuth flow.

## Step 3: List Your Connections

### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: {
    'x-publora-key': 'YOUR_API_KEY'
  }
});
const data = await response.json();
console.log(data.connections);
// [{ platformId: "twitter-123456", username: "@you", displayName: "Your Name" }, ...]
```

### Python (requests)

```python
import requests

response = requests.get(
    'https://api.publora.com/api/v1/platform-connections',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
connections = response.json()['connections']
for conn in connections:
    print(f"{conn['displayName']} ({conn['platformId']})")
```

### cURL

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY"
```

### Node.js (axios)

```javascript
const axios = require('axios');

const { data } = await axios.get(
  'https://api.publora.com/api/v1/platform-connections',
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
console.log(data.connections);
```

## Step 4: Create Your First Post

### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Hello from Publora API! 🚀',
    platforms: ['twitter-123456789', 'linkedin-ABC123'],
    scheduledTime: '2026-03-01T14:00:00.000Z'
  })
});
const data = await response.json();
console.log(data.postGroupId); // "507f1f77bcf86cd799439011"
```

### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Hello from Publora API! 🚀',
        'platforms': ['twitter-123456789', 'linkedin-ABC123'],
        'scheduledTime': '2026-03-01T14:00:00.000Z'
    }
)
print(response.json()['postGroupId'])
```

### cURL

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Hello from Publora API! 🚀",
    "platforms": ["twitter-123456789", "linkedin-ABC123"],
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
```

### Node.js (axios)

```javascript
const axios = require('axios');

const { data } = await axios.post(
  'https://api.publora.com/api/v1/create-post',
  {
    content: 'Hello from Publora API! 🚀',
    platforms: ['twitter-123456789', 'linkedin-ABC123'],
    scheduledTime: '2026-03-01T14:00:00.000Z'
  },
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
console.log(data.postGroupId);
```

## Step 5: Check Post Status

### JavaScript (fetch)

```javascript
const response = await fetch(
  `https://api.publora.com/api/v1/get-post/${postGroupId}`,
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
const data = await response.json();
// data.posts[0].status: "scheduled" | "published" | "failed"
console.log(data.posts.map(p => `${p.platform}: ${p.status}`));
```

### Python (requests)

```python
response = requests.get(
    f'https://api.publora.com/api/v1/get-post/{post_group_id}',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
for post in response.json()['posts']:
    print(f"{post['platform']}: {post['status']}")
```

### cURL

```bash
curl https://api.publora.com/api/v1/get-post/507f1f77bcf86cd799439011 \
  -H "x-publora-key: YOUR_API_KEY"
```

## What's Next?

- [Upload images and videos](guides/media-uploads.md)
- [Post to specific platforms](guides/cross-platform.md)
- [LinkedIn analytics](endpoints/linkedin-statistics.md)
- [Workspace / B2B API](guides/workspace.md)
- [Error handling](guides/error-handling.md)


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
