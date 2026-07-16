# How to Retrieve Connected Social Media Accounts with Publora

List all connected social media accounts for a user, including token health status and platform metadata.

## Overview

The Publora API provides a single endpoint to retrieve all connected social media accounts. Each connection includes the `platformId` needed for creating posts, along with token expiration status and last posting activity.

## API Reference

**Endpoint:**
```
GET https://api.publora.com/api/v1/platform-connections
```

**Headers:**
```
x-publora-key: YOUR_API_KEY
```

**Success Response (200):**
```json
{
  "success": true,
  "connections": [
    {
      "platformId": "twitter-123456789",
      "username": "@yourhandle",
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
    }
  ]
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `platformId` | string | Unique ID for creating posts (e.g., `twitter-123456789`) |
| `username` | string | Platform username or handle |
| `displayName` | string | Display name on the platform |
| `profileImageUrl` | string | Profile image URL |
| `profileUrl` | string/null | URL to profile page (null if unavailable) |
| `accessTokenExpiresAt` | string/null | ISO 8601 expiration timestamp (null = no expiration) |
| `tokenStatus` | string | Token health: `valid`, `expiring_soon`, `expired`, `unknown` |
| `tokenExpiresIn` | string/null | Human-readable time until expiration (e.g., "7d 3h") |
| `lastSuccessfulPost` | string/null | Timestamp of last successful post |
| `lastError` | object/null | Last posting error with `message` and `occurredAt` |

## Token Status Values

| Status | Meaning | Action Required |
|--------|---------|-----------------|
| `valid` | Credential is usable. Facebook, Twitter/X, and Mastodon are treated as non-expiring; YouTube/TikTok have no stored refresh expiry or it is at least 30 days away; other expiring OAuth credentials have at least 7 days left. A Bluesky row with a stored username is reported as valid by this endpoint, which does not re-check the app password. | None |
| `expiring_soon` | YouTube/TikTok refresh credential expires in under 30 days, or another expiring OAuth access token expires in under 7 days | Reconnect soon |
| `expired` | Token has expired | Reconnect required |
| `unknown` | Stored expiry data is malformed; for Bluesky in this endpoint, the projected connection has no username. Missing YouTube/TikTok `refreshTokenExpiresAt` is `valid`. | Inspect the connection and reconnect if necessary |

## Platform ID Formats

| Platform | Format | Example |
|----------|--------|---------|
| X / Twitter | `twitter-{id}` | `twitter-123456789` |
| LinkedIn | `linkedin-{id}` | `linkedin-Tz9W5i6ZYG` |
| Instagram | `instagram-{id}` | `instagram-17841412345678` |
| Threads | `threads-{id}` | `threads-17841412345678` |
| TikTok | `tiktok-{id}` | `tiktok-7123456789` |
| YouTube | `youtube-{id}` | `youtube-UCxxxxxxxxxxxx` |
| Facebook | `facebook-{id}` | `facebook-112233445566` |
| Bluesky | `bluesky-{did}` | `bluesky-did:plc:abc123` |
| Mastodon | `mastodon-{id}` | `mastodon-109876543210` |
| Telegram | `telegram-{id}` | `telegram--1001234567890` |

## Complete JavaScript Implementation

```javascript
const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY;
const BASE_URL = 'https://api.publora.com/api/v1';

/**
 * Custom error class for Publora API errors
 */
class PubloraError extends Error {
  constructor(message, statusCode, response) {
    super(message);
    this.name = 'PubloraError';
    this.statusCode = statusCode;
    this.response = response;
  }
}

/**
 * Retrieve all connected social media accounts
 * @returns {Promise<Array>} Array of connection objects
 * @throws {PubloraError} If the API request fails
 */
async function getConnectedAccounts() {
  const response = await fetch(`${BASE_URL}/platform-connections`, {
    headers: {
      'x-publora-key': PUBLORA_API_KEY
    }
  });

  // Handle non-JSON responses
  let data;
  try {
    data = await response.json();
  } catch (e) {
    throw new PubloraError(
      `Unexpected response: ${response.statusText}`,
      response.status,
      null
    );
  }

  // Handle API errors
  if (!response.ok) {
    let errorMessage = data.error || `HTTP ${response.status}`;

    if (response.status === 401) {
      errorMessage = 'Invalid API key. Check your x-publora-key header.';
    } else if (response.status === 500) {
      errorMessage = 'Server error while fetching connections.';
    }

    throw new PubloraError(errorMessage, response.status, data);
  }

  return data.connections;
}

/**
 * Get connections filtered by platform type
 * @param {string} platform - Platform name (twitter, linkedin, instagram, etc.)
 * @returns {Promise<Array>} Filtered connections
 */
async function getConnectionsByPlatform(platform) {
  const connections = await getConnectedAccounts();
  return connections.filter(c => c.platformId.startsWith(`${platform}-`));
}

/**
 * Get all platform IDs for cross-platform posting
 * @returns {Promise<string[]>} Array of platformIds
 */
async function getAllPlatformIds() {
  const connections = await getConnectedAccounts();
  return connections.map(c => c.platformId);
}

/**
 * Check for expiring or expired tokens
 * @returns {Promise<{expiring: Array, expired: Array}>} Connections needing attention
 */
async function checkTokenHealth() {
  const connections = await getConnectedAccounts();

  const expiring = connections.filter(c => c.tokenStatus === 'expiring_soon');
  const expired = connections.filter(c => c.tokenStatus === 'expired');

  return { expiring, expired };
}

/**
 * Build a map of platform type to connection IDs
 * @returns {Promise<Object>} Map like { twitter: ["twitter-123"], linkedin: ["linkedin-ABC"] }
 */
async function getConnectionsByType() {
  const connections = await getConnectedAccounts();
  const platformMap = {};

  for (const conn of connections) {
    const platform = conn.platformId.split('-')[0];
    if (!platformMap[platform]) {
      platformMap[platform] = [];
    }
    platformMap[platform].push(conn.platformId);
  }

  return platformMap;
}

// ============ USAGE EXAMPLES ============

// Example 1: List all connections
async function listAllConnections() {
  try {
    const connections = await getConnectedAccounts();

    console.log(`Found ${connections.length} connected accounts:`);
    for (const conn of connections) {
      console.log(`  ${conn.platformId}: ${conn.displayName} (@${conn.username})`);
    }

    return connections;
  } catch (error) {
    console.error('Failed to fetch connections:', error.message);
    throw error;
  }
}

// Example 2: Get platform IDs for posting
async function prepareForCrossPost() {
  try {
    const platformIds = await getAllPlatformIds();

    if (platformIds.length === 0) {
      throw new Error('No connected accounts. Connect accounts in the Publora dashboard.');
    }

    console.log('Ready to post to:', platformIds.join(', '));
    return platformIds;
  } catch (error) {
    console.error('Error:', error.message);
    throw error;
  }
}

// Example 3: Monitor token expiry
async function monitorTokens() {
  try {
    const { expiring, expired } = await checkTokenHealth();

    if (expired.length > 0) {
      console.warn('EXPIRED TOKENS - Reconnect required:');
      expired.forEach(c => console.warn(`  ${c.platformId}`));
    }

    if (expiring.length > 0) {
      console.warn('EXPIRING SOON - Reconnect recommended:');
      expiring.forEach(c => console.warn(`  ${c.platformId} - ${c.tokenExpiresIn}`));
    }

    if (expired.length === 0 && expiring.length === 0) {
      console.log('All tokens healthy');
    }

    return { expiring, expired };
  } catch (error) {
    console.error('Error checking tokens:', error.message);
    throw error;
  }
}

// Example 4: Find accounts with recent errors
async function findAccountsWithErrors() {
  const connections = await getConnectedAccounts();
  return connections.filter(c => c.lastError !== null);
}

// Run examples
const connections = await listAllConnections();
const platformIds = await prepareForCrossPost();
await monitorTokens();
```

## Complete Python Implementation

```python
import os
import requests
from typing import List, Dict, Optional
from dataclasses import dataclass

PUBLORA_API_KEY = os.environ.get('PUBLORA_API_KEY')
BASE_URL = 'https://api.publora.com/api/v1'


class PubloraError(Exception):
    """Custom exception for Publora API errors."""
    def __init__(self, message: str, status_code: int = None, response: dict = None):
        super().__init__(message)
        self.status_code = status_code
        self.response = response


@dataclass
class TokenHealth:
    """Token health check results."""
    expiring: List[Dict]
    expired: List[Dict]


def get_connected_accounts() -> List[Dict]:
    """
    Retrieve all connected social media accounts.

    Returns:
        list: Array of connection objects

    Raises:
        PubloraError: If the API request fails
    """
    response = requests.get(
        f'{BASE_URL}/platform-connections',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    # Handle non-JSON responses
    try:
        data = response.json()
    except ValueError:
        raise PubloraError(
            f'Unexpected response: {response.text}',
            response.status_code,
            None
        )

    # Handle API errors
    if not response.ok:
        error_message = data.get('error', f'HTTP {response.status_code}')

        if response.status_code == 401:
            error_message = 'Invalid API key. Check your x-publora-key header.'
        elif response.status_code == 500:
            error_message = 'Server error while fetching connections.'

        raise PubloraError(error_message, response.status_code, data)

    return data['connections']


def get_connections_by_platform(platform: str) -> List[Dict]:
    """
    Get connections filtered by platform type.

    Args:
        platform: Platform name (twitter, linkedin, instagram, etc.)

    Returns:
        list: Filtered connections
    """
    connections = get_connected_accounts()
    return [c for c in connections if c['platformId'].startswith(f'{platform}-')]


def get_all_platform_ids() -> List[str]:
    """
    Get all platform IDs for cross-platform posting.

    Returns:
        list: Array of platformIds
    """
    connections = get_connected_accounts()
    return [c['platformId'] for c in connections]


def check_token_health() -> TokenHealth:
    """
    Check for expiring or expired tokens.

    Returns:
        TokenHealth: Connections needing attention
    """
    connections = get_connected_accounts()

    expiring = [c for c in connections if c.get('tokenStatus') == 'expiring_soon']
    expired = [c for c in connections if c.get('tokenStatus') == 'expired']

    return TokenHealth(expiring=expiring, expired=expired)


def get_connections_by_type() -> Dict[str, List[str]]:
    """
    Build a map of platform type to connection IDs.

    Returns:
        dict: Map like { 'twitter': ['twitter-123'], 'linkedin': ['linkedin-ABC'] }
    """
    connections = get_connected_accounts()
    platform_map = {}

    for conn in connections:
        platform = conn['platformId'].split('-')[0]
        if platform not in platform_map:
            platform_map[platform] = []
        platform_map[platform].append(conn['platformId'])

    return platform_map


# ============ USAGE EXAMPLES ============

def list_all_connections():
    """List all connected accounts."""
    try:
        connections = get_connected_accounts()

        print(f"Found {len(connections)} connected accounts:")
        for conn in connections:
            print(f"  {conn['platformId']}: {conn['displayName']} (@{conn['username']})")

        return connections
    except PubloraError as e:
        print(f"Failed to fetch connections: {e}")
        raise


def prepare_for_cross_post() -> List[str]:
    """Get platform IDs for posting."""
    try:
        platform_ids = get_all_platform_ids()

        if not platform_ids:
            raise ValueError('No connected accounts. Connect accounts in the Publora dashboard.')

        print(f"Ready to post to: {', '.join(platform_ids)}")
        return platform_ids
    except (PubloraError, ValueError) as e:
        print(f"Error: {e}")
        raise


def monitor_tokens():
    """Monitor token expiry."""
    try:
        health = check_token_health()

        if health.expired:
            print('EXPIRED TOKENS - Reconnect required:')
            for c in health.expired:
                print(f"  {c['platformId']}")

        if health.expiring:
            print('EXPIRING SOON - Reconnect recommended:')
            for c in health.expiring:
                print(f"  {c['platformId']} - {c.get('tokenExpiresIn', 'unknown')}")

        if not health.expired and not health.expiring:
            print('All tokens healthy')

        return health
    except PubloraError as e:
        print(f"Error checking tokens: {e}")
        raise


def find_accounts_with_errors() -> List[Dict]:
    """Find accounts with recent posting errors."""
    connections = get_connected_accounts()
    return [c for c in connections if c.get('lastError') is not None]


# Run examples
if __name__ == '__main__':
    connections = list_all_connections()
    platform_ids = prepare_for_cross_post()
    monitor_tokens()
```

## cURL Examples

### List all connections

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY"
```

### With jq filtering

```bash
# Get just platform IDs
curl -s https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY" \
  | jq -r '.connections[].platformId'

# Get Twitter accounts only
curl -s https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY" \
  | jq '.connections | map(select(.platformId | startswith("twitter-")))'

# Check for expiring tokens
curl -s https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY" \
  | jq '.connections | map(select(.tokenStatus == "expiring_soon" or .tokenStatus == "expired"))'
```

## Common Use Cases

### Get platform IDs for creating a post

```javascript
// Step 1: Get all platform IDs
const connections = await getConnectedAccounts();
const platformIds = connections.map(c => c.platformId);

// Step 2: Create a post to all platforms
const response = await fetch(`${BASE_URL}/create-post`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
  },
  body: JSON.stringify({
    content: 'Hello from all my social accounts!',
    platforms: platformIds  // Post to ALL connected accounts
  })
});
```

### Post to specific platform types only

```javascript
// Post only to Twitter and LinkedIn
const connections = await getConnectedAccounts();
const selectedPlatforms = connections
  .filter(c => c.platformId.startsWith('twitter-') || c.platformId.startsWith('linkedin-'))
  .map(c => c.platformId);

await createPost(content, selectedPlatforms);
```

### Validate connections before scheduling

```javascript
async function validateBeforeScheduling(requiredPlatforms) {
  const connections = await getConnectedAccounts();
  const available = new Set(connections.map(c => c.platformId.split('-')[0]));

  const missing = requiredPlatforms.filter(p => !available.has(p));

  if (missing.length > 0) {
    throw new Error(`Missing connections for: ${missing.join(', ')}`);
  }

  // Check for expired tokens
  const expired = connections.filter(c => c.tokenStatus === 'expired');
  if (expired.length > 0) {
    throw new Error(`Expired tokens: ${expired.map(c => c.platformId).join(', ')}`);
  }

  return true;
}

// Usage
await validateBeforeScheduling(['twitter', 'linkedin']);
```

## Error Reference

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` | Check API key |
| 500 | `"Failed to fetch platform connections"` | Server error | Retry the request |

## Important Notes

### No Connections

If the response returns an empty `connections` array, the user needs to connect social accounts via the Publora dashboard at [app.publora.com](https://app.publora.com).

### Token Expiration

OAuth tokens for LinkedIn, Instagram, Threads, and TikTok expire periodically. Check `tokenStatus` regularly and prompt users to reconnect before tokens expire.

### Rate Limiting

This endpoint has generous rate limits. For most use cases, you can call it as needed without throttling.

## Related Guides

- [Create Post](../endpoints/create-post.md) - Create posts using platformIds
- [Test Connection](../endpoints/test-connection.md) - Validate a connection before posting
- [Workspace](workspace.md) - Manage connections for multiple users
