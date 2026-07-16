# Authentication

Publora uses **API keys** for authentication. API keys are long-lived credentials that don't expire or require refresh tokens — unlike OAuth, there's no token exchange workflow.

> **Important for AI Agents:** If you're an AI assistant helping a user with the Publora API, you cannot programmatically create accounts or obtain API keys. The user must complete these steps manually in their browser, then provide you with their API key.

## Prerequisites: Account Setup

Before you can authenticate with the API, you need:

### 1. Create a Publora Account

Sign up at [publora.com](https://publora.com) — free accounts are available.

### 2. Connect Your Social Media Accounts

In the Publora dashboard, connect the social platforms you want to post to:

1. Go to **Channels** in the sidebar
2. Click **Add Channel**
3. Select a platform (LinkedIn, X/Twitter, Instagram, etc.)
4. Complete the OAuth authorization flow in your browser
5. Repeat for each platform you want to use

**Note:** OAuth authorization happens in Publora's dashboard, not through the API. The API is for scheduling and managing posts to already-connected accounts.

### 3. Generate Your API Key

1. Go to **API** in the sidebar
2. Click **Generate API Key**
3. **Copy immediately** — the full key is shown only once

> **Tip:** Click **MCP** in the sidebar for MCP-specific setup instructions.

## Pricing Plans

| Plan | Price | Posts/Month | Accounts | Platforms | Video Upload |
|------|-------|-------------|----------|-----------|--------------|
| **Starter** | Free | 15 | 3 | All 10 | 50MB |
| **Pro** | $2.99/account | 100/account | Unlimited | All platforms | 100MB |
| **Premium** | $5.99/account | 500/account | Unlimited | All platforms | 250MB |

- **Starter** is free forever — great for trying the API
- **Pro** and **Premium** use per-account pricing — add as many social accounts as you need
- View full details at [publora.com/pricing](https://publora.com/pricing)

## API Keys vs OAuth Tokens

| Feature | Publora API Keys | OAuth Tokens |
|---------|------------------|--------------|
| Expiration | Never expires | Typically 1 hour |
| Refresh needed | No | Yes (refresh token flow) |
| How to get | Dashboard → API | OAuth authorization flow |
| Format | Dashboard multi-key: `sk_mrmbzomn_1a2b3c4d.964793af0123456789abcdef0123456789abcdef0123456789ab`; single-current `User.apiKey`: `sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab` | `eyJhbG...` (JWT) |

**Key point:** You can generate up to 10 active API keys from the dashboard. Each key works independently and never expires.

## Getting Your API Key

Once you have an account with connected social platforms:

1. Sign in at [publora.com](https://publora.com)
2. Go to **API** in the sidebar
3. Click **Generate API Key**
4. **Copy immediately** — the full key is shown only once

### Why Dashboard Multi-Key Generation Is Session-Only

The dashboard multi-key endpoint (`POST /auth/api-keys`) requires **dashboard session authentication**, not API-key authentication. An existing key therefore cannot mint another named dashboard key for its owner.

There is one scoped workspace exception: an authorized workspace-owner key can call `POST /api/v1/workspace/users/:userId/api-key` for an already managed user. That endpoint generates or replaces the managed user's single current `User.apiKey`; it does not create another owner/dashboard multi-key record.

This is intentional for security reasons:

| Concern | Why Session-Auth-Only |
|---------|-------------------|
| **Key theft prevention** | API-key auth cannot mint additional dashboard multi-key records for its owner; the workspace exception is restricted to an already managed user and replaces that user's current key |
| **Human verification** | Dashboard login ensures a human authorized the key |
| **Audit trail** | All key generation is logged with user/IP information |
| **Accidental exposure** | Prevents automated systems from creating excess keys |

**For automation and CI/CD:** Store your API key in environment variables or secrets managers (AWS Secrets Manager, HashiCorp Vault, GitHub Secrets). The key never expires, so you only need to set it up once.

```bash
# GitHub Actions secret
gh secret set PUBLORA_API_KEY

# AWS Secrets Manager
aws secretsmanager create-secret --name publora-api-key --secret-string "sk_..."

# Kubernetes secret
kubectl create secret generic publora --from-literal=api-key="sk_..."
```

### API Key Management Endpoints

These endpoints manage API keys and require **dashboard session authentication** (not API key auth). They power the dashboard UI and cannot be called with an API key.

| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/auth/api-keys` | List all API keys for the authenticated user |
| POST | `/auth/api-keys` | Create a new API key |
| PATCH | `/auth/api-keys/:keyId` | Update an API key (e.g., rename) |
| DELETE | `/auth/api-keys/:keyId` | Revoke (soft-delete) an API key |

**Response formats:**

- **POST** `/auth/api-keys` (Create):
  ```json
  {
    "message": "API key created successfully",
    "apiKey": {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
      "name": "My Key",
      "keyPrefix": "sk_mrmbzomn_1a2b3c4d",
      "createdAt": "2026-02-22T10:00:00.000Z",
      "lastUsedAt": null,
      "rawKey": "sk_mrmbzomn_1a2b3c4d.964793af0123456789abcdef0123456789abcdef0123456789ab"
    }
  }
  ```
  > **Important:** `rawKey` is only returned once at creation time. Store it immediately.

- **GET** `/auth/api-keys` (List):
  ```json
  {
    "apiKeys": [
      {
        "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
        "name": "My Key",
        "keyPrefix": "sk_mrmbzomn_1a2b3c4d",
        "createdAt": "2026-02-22T10:00:00.000Z",
        "lastUsedAt": "2026-02-23T14:30:00.000Z"
      }
    ]
  }
  ```

- **DELETE** `/auth/api-keys/:keyId` (Revoke):
  ```json
  { "message": "API key revoked successfully" }
  ```

- **PATCH** `/auth/api-keys/:keyId` (Update):
  ```json
  {
    "message": "API key updated successfully",
    "apiKey": {
      "_id": "65f8a1b2c3d4e5f6a7b8c9d0",
      "name": "Renamed Key",
      "keyPrefix": "sk_mrmbzomn_1a2b3c4d",
      "createdAt": "2026-02-22T10:00:00.000Z",
      "lastUsedAt": "2026-02-23T14:30:00.000Z"
    }
  }
  ```

**Key name handling:**
- Names are HTML-sanitized to prevent XSS
- If no name is provided (or the name is empty), the key defaults to `"Default"`
- Maximum name length: 100 characters

### Key Format

Publora currently mints two API-key shapes:

```text
# Dashboard key (POST /auth/api-keys)
sk_mrmbzomn_1a2b3c4d.964793af0123456789abcdef0123456789abcdef0123456789ab

# Single-current User.apiKey (session `/auth/generate-api-key` or managed-user workspace endpoint)
sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab
```

- Both start with `sk_`, contain a base36 timestamp, use one dot separator, and end with 52 random hex characters.
- Dashboard keys add an 8-hex collision-resistant segment before the dot: `sk_<timestamp_base36>_<8_hex>.<52_hex>` (about 73 characters today).
- Single-current `User.apiKey` keys use `sk_<timestamp_base36>.<52_hex>` (64 characters today). They are produced both by session-authenticated `/auth/generate-api-key` for the current user and by the API-key-authenticated managed-user workspace endpoint.
- Exact total length varies as the base36 timestamp grows. The authentication shape gate accepts 20–200 URL-safe characters, requires `sk_`, and requires a dot; the stored hash remains authoritative.
- **Dashboard keys:** Maximum **10 active keys** per user; each dashboard key has a name with a 100-character maximum.
- **Single-current `User.apiKey`:** Each user has one stored current key in this form. Session generation replaces the caller's key; workspace generation replaces the managed user's key via `$set`. Neither adds a named dashboard multi-key record.

> **Note:** The middleware enforces the broad shape above before its legacy-user fallback. A key without `sk_` or a dot is rejected even if an old user record exists; do not rely on pre-format keys.

### Key Validation (Before Making Requests)

Validate your key format before making API calls:

```javascript
function isValidPubloraKey(key) {
  return typeof key === 'string' &&
    key.length >= 20 &&
    key.length <= 200 &&
    key.startsWith('sk_') &&
    key.includes('.') &&
    /^[A-Za-z0-9._-]+$/.test(key);
}

// Usage
const apiKey = process.env.PUBLORA_API_KEY;
if (!isValidPubloraKey(apiKey)) {
  throw new Error('Invalid PUBLORA_API_KEY format');
}
```

```python
import re

def is_valid_publora_key(key):
    """Validate Publora API key format."""
    return (
        isinstance(key, str)
        and 20 <= len(key) <= 200
        and key.startswith('sk_')
        and '.' in key
        and re.fullmatch(r'[A-Za-z0-9._-]+', key) is not None
    )

# Usage
api_key = os.environ.get('PUBLORA_API_KEY')
if not is_valid_publora_key(api_key):
    raise ValueError('Invalid PUBLORA_API_KEY format')
```

## Authentication by Interface

Publora provides two interfaces that use the **same API key**. REST requires its custom header; MCP recommends Bearer authentication and also accepts the REST-style header:

| Interface | Header Format | Use Case |
|-----------|---------------|----------|
| REST API | `x-publora-key: sk_...` | Direct HTTP requests |
| MCP Server | `Authorization: Bearer sk_...` (recommended) or `x-publora-key: sk_...` | AI assistants (Claude, Cursor) |

### REST API Authentication

For direct HTTP calls to `api.publora.com`:

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab"
```

```javascript
const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: {
    'x-publora-key': process.env.PUBLORA_API_KEY
  }
});
```

```python
response = requests.get(
    'https://api.publora.com/api/v1/platform-connections',
    headers={'x-publora-key': os.environ['PUBLORA_API_KEY']}
)
```

### MCP Authentication

For MCP clients connecting to `mcp.publora.com`:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab"
      }
    }
  }
}
```

**Why recommend Bearer for MCP?** MCP clients commonly expect the standard `Authorization: Bearer` format. The MCP server also accepts `x-publora-key` as a fallback. Direct REST requests accept only `x-publora-key`.

### MCP Client Identification

Publora's MCP server sets `x-publora-client: mcp` on its internal REST requests to the backend. This triggers an `mcpAccess` entitlement check. If the account does not have MCP access enabled, the backend responds with `403 "MCP access is not enabled for this account"`.

Claude Desktop, Cursor, and other MCP clients do not send this REST header; they authenticate to `mcp.publora.com`, and Publora's MCP server adds it downstream. A direct REST caller may also set `x-publora-client`; the value `mcp` opts that request into the same entitlement check.

The backend stores any non-empty `x-publora-client` value verbatim as `req.apiUser.client`; when the header is absent it uses `"api"`.

### Internal Request Context (`req.apiUser`)

After successful authentication, the middleware attaches a `req.apiUser` object with the following fields:

| Field | Type | Description |
|-------|------|-------------|
| `userId` | `ObjectId` | The effective user ID (target user if workspace, otherwise key owner) |
| `ownerId` | `ObjectId` | The billing owner's user ID (may differ from the direct key owner for managed workspace users, resolved via `resolveWorkspaceEntitlementsByActor`) |
| `ownerUser` | `Object` | Subset of the `actorUser` document (the user whose API key was used, resolved via workspace entitlements) containing `{ _id, permissions, isAdmin }` |
| `billingOwnerUser` | `Object` | The user responsible for billing. Contains `{ _id, permissions, isAdmin, entitlements }`. May differ from `ownerUser` in workspace setups |
| `keyPrefix` | `string` | The portion before the required dot separator, retained for lookup/log context without exposing the secret suffix |
| `isWorkspace` | `boolean` | `true` if acting on behalf of a managed user via `x-publora-user-id` |
| `client` | `string` | Arbitrary client identifier copied from `x-publora-client`; `"api"` when absent. Publora's MCP server sends `"mcp"` |
| `entitlements` | `Object` | Feature flags and plan capabilities for the billing owner |

> **Note:** These fields are internal to the server — they are not returned in API responses. They are documented here for contributors and advanced integrators who may encounter them in error messages or logs.

## Complete Authentication Workflow

### Step 1: Store Your Key Securely

```bash
# Add to ~/.bashrc or ~/.zshrc
export PUBLORA_API_KEY="sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab"
```

Or use a `.env` file (add to `.gitignore`):

```bash
# .env
PUBLORA_API_KEY=sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab
```

### Step 2: Verify Your Key Works

Test authentication by fetching your connected platforms:

```javascript
async function verifyAuthentication() {
  const apiKey = process.env.PUBLORA_API_KEY;

  // Validate format first
  if (!apiKey?.startsWith('sk_')) {
    console.error('Error: API key must start with sk_');
    console.error('Get your key at: https://publora.com → API');
    return false;
  }

  try {
    const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
      headers: { 'x-publora-key': apiKey }
    });

    if (response.status === 401) {
      console.error('Error: Invalid API key');
      console.error('Check your key or generate a new one at publora.com');
      return false;
    }

    if (!response.ok) {
      console.error(`Error: HTTP ${response.status}`);
      return false;
    }

    const data = await response.json();
    console.log(`✓ Authenticated successfully`);
    console.log(`✓ ${data.connections.length} connected platform(s)`);
    return true;
  } catch (error) {
    console.error('Error: Connection failed -', error.message);
    return false;
  }
}
```

```python
import os
import requests

def verify_authentication():
    """Verify API key is valid and working."""
    api_key = os.environ.get('PUBLORA_API_KEY')

    # Validate format first
    if not api_key or not api_key.startswith('sk_'):
        print('Error: API key must start with sk_')
        print('Get your key at: https://publora.com → API')
        return False

    try:
        response = requests.get(
            'https://api.publora.com/api/v1/platform-connections',
            headers={'x-publora-key': api_key}
        )

        if response.status_code == 401:
            print('Error: Invalid API key')
            print('Check your key or generate a new one at publora.com')
            return False

        response.raise_for_status()
        data = response.json()
        print(f'✓ Authenticated successfully')
        print(f"✓ {len(data['connections'])} connected platform(s)")
        return True

    except requests.RequestException as e:
        print(f'Error: Connection failed - {e}')
        return False

# Run verification
verify_authentication()
```

### Step 3: Make Your First API Call

```javascript
// After verification succeeds, make API calls
const response = await fetch('https://api.publora.com/api/v1/list-posts', {
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
});
const { posts } = await response.json();
console.log(`Found ${posts.length} posts`);
```

## Workspace Authentication

For B2B workspaces managing multiple users, add the user ID header. There is no separate "workspace API key" — you use the same API key. The middleware checks `actorUser` (resolved via workspace entitlements), which may differ from the direct key owner for managed users.

| Header | Value | Purpose |
|--------|-------|---------|
| `x-publora-key` | Your API key | Authenticates your account |
| `x-publora-user-id` | Managed user's ObjectId | Specifies which user to act as |

The target user must have their `parentUser` field set to the API key owner. This `parentUser` relationship is what authorizes the key owner to act on behalf of that user. Without it, the request will be rejected with a 403 error.

> **Note:** Passing your own user ID as `x-publora-user-id` is a no-op — the middleware detects the self-reference and skips the workspace/parentUser check entirely. The request proceeds as if the header were not set.

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': process.env.PUBLORA_API_KEY,
    'x-publora-user-id': '507f1f77bcf86cd799439011'  // Managed user (must have parentUser set to key owner)
  },
  body: JSON.stringify({
    content: 'Posted on behalf of managed user',
    platforms: ['twitter-123456']
  })
});
```

## Error Handling

### Authentication Errors

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 400 | `"Invalid x-publora-user-id"` | `x-publora-user-id` header is not a valid ObjectId | Provide a valid 24-character hex ObjectId |
| 401 | `"API key is required"` | Missing `x-publora-key` header | Include the `x-publora-key` header with your API key |
| 401 | `"Invalid API key"` | Key is wrong or has been revoked | Generate a new key at API in sidebar |
| 401 | `"Invalid API key owner"` | User associated with the key could not be found | Contact support; the key owner account may be deleted |
| 403 | `"API access is not enabled for this account"` | Account lacks the API access entitlement | Contact support if you expect API access |
| 403 | `"Your current plan does not include API access"` | API access is disabled for this account (returned by key management endpoints) | Contact support if you expect API access |
| 403 | `"MCP access is not enabled for this account"` | Account lacks MCP access entitlement (sent via MCP client) | Upgrade your plan to include MCP access |
| 403 | `"Workspace access is not enabled for this key"` | Used `x-publora-user-id` but key owner does not have `workspacesEnabled` | Enable workspace access on your account or remove the header. **Note:** Admin users (`isAdmin: true`) automatically bypass this check and have workspace access regardless of the `workspacesEnabled` permission |
| 403 | `"User is not managed by key"` | Target user's `parentUser` is not set to the key owner | Ensure the managed user has `parentUser` set to your user ID |
| 400 | `"Maximum of 10 active API keys allowed"` | Attempted to create an API key when 10 active keys already exist | Delete an existing key before creating a new one |
| 400 | `"Name must be a string with maximum 100 characters"` | POST `/auth/api-keys` — name is not a string or exceeds 100 characters | Provide a valid name under 100 characters |
| 404 | `"User not found"` | POST `/auth/api-keys` — authenticated user could not be found in the database | Contact support; the account may be deleted |
| 400 | `"Name is required"` | PATCH `/auth/api-keys/:keyId` — name field is missing or empty | Provide a non-empty name |
| 400 | `"Name must be maximum 100 characters"` | PATCH `/auth/api-keys/:keyId` — name exceeds 100 characters | Shorten the name to 100 characters or fewer |
| 400 | `"Invalid key ID format"` | PATCH or DELETE `/auth/api-keys/:keyId` — keyId is not a valid ObjectId | Provide a valid 24-character hex ObjectId |
| 404 | `"API key not found"` | DELETE or PATCH `/auth/api-keys/:keyId` — the specified key doesn't exist or has been revoked | Verify the key ID is correct and the key has not already been deleted |
| 500 | `"Internal server error"` | Unexpected error during authentication middleware | Retry the request; if persistent, contact support |

### Handling Auth Errors in Code

```javascript
async function publoraRequest(endpoint, options = {}) {
  const apiKey = process.env.PUBLORA_API_KEY;

  if (!apiKey?.startsWith('sk_')) {
    throw new Error('PUBLORA_API_KEY not set or invalid format');
  }

  const response = await fetch(`https://api.publora.com/api/v1${endpoint}`, {
    ...options,
    headers: {
      'x-publora-key': apiKey,
      'Content-Type': 'application/json',
      ...options.headers
    }
  });

  if (response.status === 401) {
    throw new Error('Invalid API key. Generate a new one at publora.com → API');
  }

  if (response.status === 403) {
    const data = await response.json();
    if (data.error?.includes('API access is not enabled')) {
      throw new Error('API access not enabled. Upgrade at publora.com/pricing');
    }
    throw new Error(data.error || 'Access denied');
  }

  if (!response.ok) {
    const data = await response.json();
    throw new Error(data.error || `HTTP ${response.status}`);
  }

  return response.json();
}
```

```python
class PubloraAuthError(Exception):
    """Raised when authentication fails."""
    pass

def publora_request(endpoint, method='GET', **kwargs):
    """Make authenticated request with proper error handling."""
    api_key = os.environ.get('PUBLORA_API_KEY')

    if not api_key or not api_key.startswith('sk_'):
        raise PubloraAuthError('PUBLORA_API_KEY not set or invalid format')

    response = requests.request(
        method,
        f'https://api.publora.com/api/v1{endpoint}',
        headers={
            'x-publora-key': api_key,
            'Content-Type': 'application/json'
        },
        **kwargs
    )

    if response.status_code == 401:
        raise PubloraAuthError('Invalid API key. Generate a new one at publora.com → API')

    if response.status_code == 403:
        data = response.json()
        if 'API access is not enabled' in data.get('error', ''):
            raise PubloraAuthError('API access not enabled. Upgrade at publora.com/pricing')
        raise PubloraAuthError(data.get('error', 'Access denied'))

    response.raise_for_status()
    return response.json()
```

## Retry Logic with Exponential Backoff

For production applications, implement retry logic:

```javascript
async function publoraRequestWithRetry(endpoint, options = {}, maxRetries = 3) {
  const apiKey = process.env.PUBLORA_API_KEY;

  if (!apiKey?.startsWith('sk_')) {
    throw new Error('PUBLORA_API_KEY not set or invalid format');
  }

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(`https://api.publora.com/api/v1${endpoint}`, {
        ...options,
        headers: {
          'x-publora-key': apiKey,
          'Content-Type': 'application/json',
          ...options.headers
        }
      });

      // Don't retry auth errors - they won't succeed
      if (response.status === 401 || response.status === 403) {
        const data = await response.json();
        throw new Error(data.error || `Auth error: ${response.status}`);
      }

      // Retry on rate limit
      if (response.status === 429) {
        const retryAfter = parseInt(response.headers.get('Retry-After') || '60');
        console.log(`Rate limited. Waiting ${retryAfter}s...`);
        await new Promise(r => setTimeout(r, retryAfter * 1000));
        continue;
      }

      // Retry on server errors
      if (response.status >= 500 && attempt < maxRetries) {
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Server error. Retry ${attempt}/${maxRetries} in ${delay}ms...`);
        await new Promise(r => setTimeout(r, delay));
        continue;
      }

      if (!response.ok) {
        const data = await response.json();
        throw new Error(data.error || `HTTP ${response.status}`);
      }

      return response.json();
    } catch (error) {
      if (attempt === maxRetries) throw error;
      if (error.message.includes('Auth error')) throw error; // Don't retry auth errors
    }
  }
}
```

## Security Best Practices

### Do

- Store keys in environment variables
- Use `.env` files (added to `.gitignore`)
- Rotate keys periodically
- Use separate keys for development and production
- Validate key format before making requests

### Don't

- Hardcode keys in source code
- Commit keys to version control
- Share keys in chat, email, or public forums
- Use keys in client-side JavaScript (browsers)
- Log full API keys (mask them: `sk_mrmbzomn.9647...****`)

### Key Storage

```javascript
// Good: Environment variable
const apiKey = process.env.PUBLORA_API_KEY;

// Bad: Hardcoded
const apiKey = 'sk_mrmbzomn.964793af0123456789abcdef0123456789abcdef0123456789ab'; // Never do this!
```

### Managing API Keys

You can generate multiple API keys — useful for different environments or applications.

**If your key is compromised:**

1. Go to publora.com → **API** in the sidebar
2. Delete the compromised key
3. Click **Generate API Key** to create a new one
4. Update your environment variables with the new key

> **Note:** Key deletion is a soft-delete — it sets a `revokedAt` timestamp rather than removing the key from the database. Revoked keys immediately stop working but remain in the audit trail.

## Quick Reference

| What | Value |
|------|-------|
| REST API Base URL | `https://api.publora.com/api/v1` |
| MCP Server URL | `https://mcp.publora.com` |
| REST API Header | `x-publora-key: sk_...` |
| MCP Header | `Authorization: Bearer sk_...` (recommended) or `x-publora-key: sk_...` |
| Key Format | Dashboard multi-key: `sk_<timestamp_base36>_<8_hex>.<52_hex>`; single-current `User.apiKey`: `sk_<timestamp_base36>.<52_hex>` |
| Key Expiration | Never (until revoked) |
| Max Keys | Dashboard: 10 active multi-key records per user; `User.apiKey`: one current key per user, replaced on regeneration |
| Get Key | publora.com → API |

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
