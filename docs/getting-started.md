# Getting Started with Publora API

## Overview

Publora API lets you schedule and publish social media posts across 10 platforms from a single REST endpoint. Base URL: `https://api.publora.com`

Machine-readable API descriptions are available as [OpenAPI YAML](https://docs.publora.com/openapi.yaml) and [OpenAPI JSON](https://docs.publora.com/openapi.json).

> **For AI Agents:** You cannot programmatically create accounts or generate API keys. Your user must complete Steps 1-2 manually at [publora.com](https://publora.com), then provide you with their API key.

## Pricing

| Plan | Price | Posts/Month | Platforms |
|------|-------|-------------|-----------|
| **Starter** | Free | 15 | All 10 |
| **Pro** | $2.99/account | 100/account | All platforms |
| **Premium** | $5.99/account | 500/account | All platforms |

> **Note:** The free Starter plan **includes** full REST API and MCP access (3 connected accounts, 15 posts/month account-wide).

See full details at [publora.com/pricing](https://publora.com/pricing)

## Step 1: Sign Up and Get an API Key

1. Create an account at [publora.com](https://publora.com) (free tier available)
2. Go to **API** in the sidebar
3. Click **Generate API Key**
4. Copy the key immediately — it's shown only once

Dashboard keys look like: `sk_mrmbzomn_1a2b3c4d.964793af0123456789abcdef0123456789abcdef0123456789ab`

## Step 2: Connect Social Accounts

Connect your social media accounts via the Publora dashboard:

1. Go to **Channels** in the sidebar
2. Click **Add Channel** and select a platform
3. Complete the OAuth authorization in your browser
4. Repeat for each platform you want to post to

**Note:** Social account connections use OAuth and must be done in the dashboard — not via API. The API is for scheduling posts to already-connected accounts.

> **Tip:** Click **MCP** in the sidebar for AI assistant integration instructions.

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
const scheduledTime = new Date(Date.now() + 5 * 60 * 1000).toISOString();
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Hello from Publora API! 🚀',
    platforms: ['twitter-123456789', 'linkedin-ABC123'],
    scheduledTime
  })
});
const data = await response.json();
console.log(data.postGroupId); // "507f1f77bcf86cd799439011"
```

### Python (requests)

```python
import requests
from datetime import datetime, timedelta, timezone

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Hello from Publora API! 🚀',
        'platforms': ['twitter-123456789', 'linkedin-ABC123'],
        'scheduledTime': (datetime.now(timezone.utc) + timedelta(minutes=5)).isoformat()
    }
)
print(response.json()['postGroupId'])
```

### cURL

Replace `<FUTURE_ISO_8601_UTC>` with a UTC time at least five minutes ahead.

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Hello from Publora API! 🚀",
    "platforms": ["twitter-123456789", "linkedin-ABC123"],
    "scheduledTime": "<FUTURE_ISO_8601_UTC>"
  }'
```

### Node.js (axios)

```javascript
const axios = require('axios');
const scheduledTime = new Date(Date.now() + 5 * 60 * 1000).toISOString();

const { data } = await axios.post(
  'https://api.publora.com/api/v1/create-post',
  {
    content: 'Hello from Publora API! 🚀',
    platforms: ['twitter-123456789', 'linkedin-ABC123'],
    scheduledTime
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
