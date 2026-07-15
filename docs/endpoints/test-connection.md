# Test Connection

Validate that a platform connection is working before scheduling posts.

## Endpoint

```
POST https://api.publora.com/api/v1/test-connection/:platformId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Client identifier (e.g., `mcp`). Used for MCP access gating. |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `platformId` | string | The platform ID (e.g., `linkedin-Tz9W5i6ZYG`) |

## Response

### Successful Connection

```json
{
  "platform": "linkedin",
  "platformId": "linkedin-Tz9W5i6ZYG",
  "username": "John Doe",
  "status": "ok",
  "message": "Connected as John Doe",
  "permissions": ["w_member_social", "w_member_social_feed", "r_basicprofile", "r_member_postAnalytics", "r_member_profileAnalytics"],
  "tokenExpiresIn": "45d 3h",
  "lastSuccessfulPost": "2026-02-22T09:15:00.000Z",
  "lastError": null
}
```

### Failed Connection

```json
{
  "platform": "threads",
  "platformId": "threads-17841412345678",
  "username": "yourthreads",
  "status": "error",
  "message": "Error validating access token: Session has expired",
  "permissions": [],
  "tokenExpiresIn": "expired",
  "lastSuccessfulPost": "2026-02-15T14:30:00.000Z",
  "lastError": {
    "message": "Session has expired",
    "occurredAt": "2026-02-22T10:00:00.000Z"
  }
}
```

> **Important:** All validation results — both success and failure — return HTTP 200. The connection test outcome is indicated by the `status` field (`"ok"` or `"error"`), not the HTTP status code. Only infrastructure errors (404 connection not found, 500 server error) use non-200 status codes.

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `platform` | string | Platform name (twitter, linkedin, threads, etc.) |
| `platformId` | string | Full platform ID |
| `username` | string | Connected account username |
| `status` | string | `ok` or `error` |
| `message` | string | Human-readable status message |
| `permissions` | array | List of granted permissions/scopes |
| `tokenExpiresIn` | string/null | Time until token expiry. Possible values: `"7d 3h"` (days and hours), `"5h"` (hours only, when less than one day remains), `"< 1h"` (less than one hour remaining), `"expired"` (token already expired), or `null` (no expiration). |
| `lastSuccessfulPost` | string/null | Timestamp of last successful post |
| `lastError` | object/null | Last error details if any. When present, contains: `message` (string — error description) and `occurredAt` (string — ISO 8601 timestamp of when the error occurred). |

## Examples

### JavaScript (fetch)

```javascript
const platformId = 'linkedin-Tz9W5i6ZYG';
const response = await fetch(
  `https://api.publora.com/api/v1/test-connection/${platformId}`,
  {
    method: 'POST',
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);
const result = await response.json();

if (result.status === 'ok') {
  console.log(`✅ ${result.message}`);
  console.log(`Token expires in: ${result.tokenExpiresIn}`);
} else {
  console.log(`❌ Connection failed: ${result.message}`);
}
```

### Python (requests)

```python
import requests

platform_id = 'linkedin-Tz9W5i6ZYG'
response = requests.post(
    f'https://api.publora.com/api/v1/test-connection/{platform_id}',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)
result = response.json()

if result['status'] == 'ok':
    print(f"✅ {result['message']}")
    print(f"Permissions: {', '.join(result['permissions'])}")
else:
    print(f"❌ {result['message']}")
```

### cURL

```bash
curl -X POST https://api.publora.com/api/v1/test-connection/linkedin-Tz9W5i6ZYG \
  -H "x-publora-key: YOUR_API_KEY"
```

### Pre-flight Check Before Scheduling

```javascript
async function validateConnectionsBeforePost(platformIds) {
  const results = await Promise.all(
    platformIds.map(async (platformId) => {
      const response = await fetch(
        `https://api.publora.com/api/v1/test-connection/${platformId}`,
        {
          method: 'POST',
          headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
        }
      );
      return response.json();
    })
  );

  const failed = results.filter(r => r.status === 'error');
  if (failed.length > 0) {
    console.log('⚠️  Some connections need attention:');
    failed.forEach(r => console.log(`  - ${r.platformId}: ${r.message}`));
    return false;
  }

  console.log('✅ All connections healthy');
  return true;
}

// Usage
const platforms = ['twitter-123', 'linkedin-ABC', 'threads-456'];
const healthy = await validateConnectionsBeforePost(platforms);
if (healthy) {
  // Proceed with scheduling
}
```

### Telegram MTProto Connections

> **Note:** Telegram MTProto connections will always fail the test-connection check because the validator only checks `connectionType === "bot"` connections. MTProto connections fall through to the default failure case, returning a misleading `"Invalid bot token"` error.

### Unknown Platform

If the platform has no test-connection validator implemented (e.g., Pinterest), the endpoint returns HTTP 200 with a full response structure but error status:

```json
{
  "platform": "pinterest",
  "platformId": "pinterest-abc123",
  "username": "unknown",
  "status": "error",
  "message": "Unknown platform: pinterest",
  "permissions": [],
  "tokenExpiresIn": null,
  "lastSuccessfulPost": null,
  "lastError": null
}
```

> **Note:** The public API route returns only `{ error: "Failed to test connection" }` for 500 errors without a `message` field. The internal dashboard endpoint may include an additional `message` field in error responses.

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Invalid platformId format"` | `platformId` doesn't contain a dash or is malformed. **Note:** This endpoint uses a simple dash-split validation. `create-post` instead parses at the first dash, requires a lowercase-letter prefix, and rejects whitespace, `/`, `?`, or `#` in the non-empty ID; colons are allowed. |
| 400 | `"Invalid x-publora-user-id"` | `x-publora-user-id` header contains an invalid ObjectId |
| 401 | `"API key is required"` | Missing `x-publora-key` header |
| 401 | `"Invalid API key"` | `x-publora-key` value is not a valid key |
| 401 | `"Invalid API key owner"` | The API key's owner account could not be found |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 403 | `"MCP access is not enabled for this account"` | MCP access is not enabled when `x-publora-client: mcp` is sent |
| 403 | `"Workspace access is not enabled for this key"` | API key does not have workspace permissions |
| 403 | `"User is not managed by key"` | The `x-publora-user-id` user is not managed by this API key |
| 500 | `"Internal server error"` | Unexpected error in middleware |
| 404 | `"Connection not found"` | Invalid platformId or connection doesn't exist |
| 500 | `"Failed to test connection"` | Server error during validation |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
