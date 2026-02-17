# Platform Connections

List all connected social media accounts.

## Endpoint

```
GET https://api.publora.com/api/v1/platform-connections
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |

## Response

```json
{
  "success": true,
  "connections": [
    {
      "platformId": "twitter-123456789",
      "username": "@yourhandle",
      "displayName": "Your Name",
      "profileImageUrl": "https://pbs.twimg.com/profile_images/...",
      "accessTokenExpiresAt": null
    },
    {
      "platformId": "linkedin-Tz9W5i6ZYG",
      "username": "John Doe",
      "displayName": "John Doe",
      "profileImageUrl": "https://media.licdn.com/...",
      "accessTokenExpiresAt": "2026-05-15T10:30:00.000Z"
    },
    {
      "platformId": "instagram-17841412345678",
      "username": "yourinstagram",
      "displayName": "Your Brand",
      "profileImageUrl": "https://...",
      "accessTokenExpiresAt": "2026-04-20T08:00:00.000Z"
    }
  ]
}
```

## Connection Fields

| Field | Type | Description |
|-------|------|-------------|
| `platformId` | string | Unique ID in format `platform-id`. Use this in the `platforms` array when creating posts. |
| `username` | string | Platform username or handle |
| `displayName` | string | Display name on the platform |
| `profileImageUrl` | string | Profile image URL |
| `accessTokenExpiresAt` | string/null | Token expiration (null = no expiration) |

## Platform ID Format

The `platformId` follows the pattern `platform-id`:

| Platform | Example platformId |
|----------|-------------------|
| X / Twitter | `twitter-123456789` |
| LinkedIn | `linkedin-Tz9W5i6ZYG` |
| Instagram | `instagram-17841412345678` |
| Threads | `threads-17841412345678` |
| TikTok | `tiktok-7123456789` |
| YouTube | `youtube-UCxxxxxxxxxxxx` |
| Facebook | `facebook-112233445566` |
| Bluesky | `bluesky-did:plc:abc123` |
| Mastodon | `mastodon-109876543210` |
| Telegram | `telegram--1001234567890` |

## Examples

### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: { 'x-publora-key': 'YOUR_API_KEY' }
});
const data = await response.json();

// Filter by platform
const twitterAccounts = data.connections.filter(c => c.platformId.startsWith('twitter-'));
const linkedinAccounts = data.connections.filter(c => c.platformId.startsWith('linkedin-'));

console.log(`Twitter accounts: ${twitterAccounts.length}`);
console.log(`LinkedIn accounts: ${linkedinAccounts.length}`);
```

### Python (requests)

```python
import requests

response = requests.get(
    'https://api.publora.com/api/v1/platform-connections',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
connections = response.json()['connections']

# Get all platform IDs for cross-platform posting
all_platform_ids = [c['platformId'] for c in connections]
print(f"Connected to {len(all_platform_ids)} accounts")

# Check for expiring tokens
from datetime import datetime, timezone
for conn in connections:
    if conn.get('accessTokenExpiresAt'):
        expires = datetime.fromisoformat(conn['accessTokenExpiresAt'].replace('Z', '+00:00'))
        if expires < datetime.now(timezone.utc):
            print(f"⚠️  {conn['platformId']} token expired! Reconnect in dashboard.")
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

// Build a map of platform → connection IDs
const platforms = {};
for (const conn of data.connections) {
  const platform = conn.platformId.split('-')[0];
  if (!platforms[platform]) platforms[platform] = [];
  platforms[platform].push(conn.platformId);
}
console.log(platforms);
// { twitter: ["twitter-123"], linkedin: ["linkedin-ABC"], ... }
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |

## Connecting Accounts

Social accounts are connected via the Publora dashboard (OAuth flow). The API does not support connecting new accounts programmatically -- use the dashboard at [app.publora.com](https://app.publora.com).

For workspace users, generate a connection URL via the [Workspace API](../guides/workspace.md).


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
