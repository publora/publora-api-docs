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
| `x-publora-client` | No | Client identifier (e.g., `mcp`). Used for MCP access gating. |

> **Browser usage:** CORS permits the Publora headers above, but only requests from origins in Publora's deployment allowlist are accepted. An arbitrary integrator origin can still fail preflight. Keep the API key on a backend or serverless function; exposing it in client-side JavaScript leaks the credential.

## Response

```json
{
  "success": true,
  "connections": [
    {
      "platformId": "twitter-123456789",
      "username": "yourhandle",
      "displayName": "Your Name",
      "profileImageUrl": "https://pbs.twimg.com/profile_images/...",
      "profileUrl": "https://twitter.com/yourhandle",
      "accessTokenExpiresAt": null,
      "tokenStatus": "valid",
      "tokenExpiresIn": null,
      "lastSuccessfulPost": "2026-02-20T14:30:00.000Z",
      "lastError": null
    },
    {
      "platformId": "linkedin-Tz9W5i6ZYG",
      "username": "John Doe",
      "displayName": "John Doe",
      "profileImageUrl": "https://media.licdn.com/...",
      "profileUrl": "https://linkedin.com/in/johndoe",
      "accessTokenExpiresAt": "2026-05-15T10:30:00.000Z",
      "tokenStatus": "valid",
      "tokenExpiresIn": "82d 4h",
      "lastSuccessfulPost": "2026-02-22T09:15:00.000Z",
      "lastError": null
    },
    {
      "platformId": "instagram-17841412345678",
      "username": "yourinstagram",
      "displayName": "Your Brand",
      "profileImageUrl": "https://...",
      "profileUrl": null,
      "accessTokenExpiresAt": "2026-02-25T08:00:00.000Z",
      "tokenStatus": "expiring_soon",
      "tokenExpiresIn": "2d 12h",
      "lastSuccessfulPost": null,
      "lastError": {
        "message": "Media upload failed: Invalid image format",
        "occurredAt": "2026-02-21T16:45:00.000Z"
      }
    }
  ]
}
```

## Connection Fields

| Field | Type | Description |
|-------|------|-------------|
| `platformId` | string | Unique ID in format `platform-id`. Use this in the `platforms` array when creating posts. |
| `username` | string | Platform username or handle. Note: the stored username may or may not include a `@` prefix depending on what was saved during OAuth. |
| `displayName` | string/null | Display name on the platform. Returns `null` if not set during OAuth. |
| `profileImageUrl` | string/null | Profile image URL. Returns `null` if not available. |
| `profileUrl` | string/null | URL to the user's profile on the platform. Can be null if not available for the platform. |
| `accessTokenExpiresAt` | string/null | Token expiration timestamp (null = no expiration) |
| `tokenStatus` | string | Current credential health: `valid`, `expiring_soon`, `expired`, or `unknown`; see platform-specific rules below |
| `tokenExpiresIn` | string/null | Human-readable time until expiration. Possible formats: `"7d 3h"` (days and hours), `"5h"` (hours only, when less than one day remains), `"< 1h"` (less than one hour remaining), `"expired"` (token already expired), or `null` (no expiration). |
| `lastSuccessfulPost` | string/null | Timestamp of last successful post to this platform |
| `lastError` | object/null | Last error that occurred when posting to this platform. When present, contains: `message` (string — error description) and `occurredAt` (string — ISO 8601 timestamp of when the error occurred). |
| `subscriptionType` | string/null | Platform tier where available (X publishing recognizes exact stored values `"Premium"` and `"PremiumPlus"`). `null` when not detected. |

## Token Status Values

| Status | Meaning |
|--------|---------|
| `valid` | Credential is usable. Facebook, Twitter/X, and Mastodon are treated as non-expiring. YouTube/TikTok are also `valid` when `refreshTokenExpiresAt` is absent or at least 30 days away. For Bluesky, this endpoint's projected connection data marks a stored username as `valid`; it does not re-check the app password. |
| `expiring_soon` | YouTube/TikTok refresh token expires in under 30 days, or another OAuth platform's access token expires in under 7 days |
| `expired` | Token has expired - reconnect required |
| `unknown` | Stored expiry data is malformed. For Bluesky in this endpoint, it means the projected connection has no username. A missing YouTube/TikTok `refreshTokenExpiresAt` is `valid`, not `unknown`. |

> **Note:** Pinterest has OAuth connection routes in the dashboard, but no `test-connection` validator is implemented. Calling `test-connection` for a Pinterest connection will return `"Unknown platform: pinterest"`.

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

### With Error Handling

```javascript
async function getConnections() {
  try {
    const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
      headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
    });

    if (response.status === 401) {
      throw new Error('Invalid API key. Check your PUBLORA_API_KEY.');
    }

    if (!response.ok) {
      const data = await response.json();
      throw new Error(data.error || `HTTP ${response.status}`);
    }

    const data = await response.json();

    // Check for expiring tokens
    const now = new Date();
    for (const conn of data.connections) {
      if (conn.accessTokenExpiresAt) {
        const expires = new Date(conn.accessTokenExpiresAt);
        const daysUntilExpiry = Math.floor((expires - now) / (1000 * 60 * 60 * 24));
        if (daysUntilExpiry < 7) {
          console.warn(`⚠️  ${conn.platformId} token expires in ${daysUntilExpiry} days`);
        }
      }
    }

    return data.connections;
  } catch (error) {
    console.error('Failed to fetch connections:', error.message);
    throw error;
  }
}
```

```python
import os
import requests
from datetime import datetime, timezone

def get_connections():
    """Get all connected platforms with error handling and token expiry check."""
    try:
        response = requests.get(
            'https://api.publora.com/api/v1/platform-connections',
            headers={'x-publora-key': os.environ['PUBLORA_API_KEY']}
        )

        if response.status_code == 401:
            raise ValueError('Invalid API key. Check your PUBLORA_API_KEY.')

        response.raise_for_status()
        data = response.json()

        # Check for expiring tokens
        now = datetime.now(timezone.utc)
        for conn in data['connections']:
            if conn.get('accessTokenExpiresAt'):
                expires = datetime.fromisoformat(
                    conn['accessTokenExpiresAt'].replace('Z', '+00:00')
                )
                days_until_expiry = (expires - now).days
                if days_until_expiry < 7:
                    print(f"⚠️  {conn['platformId']} token expires in {days_until_expiry} days")

        return data['connections']

    except requests.RequestException as e:
        print(f'Failed to fetch connections: {e}')
        raise


# Usage
connections = get_connections()
platform_ids = [c['platformId'] for c in connections]
print(f"Connected to {len(platform_ids)} platforms: {platform_ids}")
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Invalid x-publora-user-id"` | `x-publora-user-id` header contains an invalid ObjectId |
| 401 | `"API key is required"` | Missing `x-publora-key` header |
| 401 | `"Invalid API key"` | `x-publora-key` value is not a valid key |
| 401 | `"Invalid API key owner"` | The API key's owner account could not be found |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 403 | `"MCP access is not enabled for this account"` | MCP access is not enabled when `x-publora-client: mcp` is sent |
| 403 | `"Workspace access is not enabled for this key"` | API key does not have workspace permissions |
| 403 | `"User is not managed by key"` | The `x-publora-user-id` user is not managed by this API key |
| 500 | `"Internal server error"` | Unexpected error in middleware |
| 500 | `"Failed to fetch platform connections"` | Server error while fetching connections |

## Connecting Accounts

Social accounts are connected via the Publora dashboard (OAuth flow). The API does not support connecting new accounts programmatically -- use the dashboard at [app.publora.com](https://app.publora.com).

For workspace users, generate a connection URL via the [Workspace API](../guides/workspace.md).

### Pinterest connection limitation

Pinterest can be connected through OAuth, but it is connect-only. The scheduler has no Pinterest dispatch branch, so do not target a Pinterest connection in create/update scheduling requests.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
