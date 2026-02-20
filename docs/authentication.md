# Authentication

Publora uses **API keys** for authentication. API keys are long-lived credentials that don't expire or require refresh tokens — unlike OAuth, there's no token exchange workflow.

## API Keys vs OAuth Tokens

| Feature | Publora API Keys | OAuth Tokens |
|---------|------------------|--------------|
| Expiration | Never expires | Typically 1 hour |
| Refresh needed | No | Yes (refresh token flow) |
| How to get | Dashboard → Settings | OAuth authorization flow |
| Format | `sk_kzq5mjw.a1b2c3...` | `eyJhbG...` (JWT) |

**Key point:** You get one API key from the dashboard and use it forever (or until you regenerate it).

## Getting Your API Key

### Step-by-Step

1. Sign in at [publora.com](https://publora.com)
2. Go to **Settings** → **API Keys**
3. Click **Generate API Key**
4. **Copy immediately** — the full key is shown only once

### Key Format

```
sk_kzq5mjw.a1b2c3d4e5f6g7h8i9j0...
```

- Starts with `sk_` prefix
- Contains a timestamp segment
- Followed by a random hex string
- Total length: ~50 characters

### Key Validation (Before Making Requests)

Validate your key format before making API calls:

```javascript
function isValidPubloraKey(key) {
  if (!key || typeof key !== 'string') return false;
  if (!key.startsWith('sk_')) return false;
  if (key.length < 20) return false;
  return true;
}

// Usage
const apiKey = process.env.PUBLORA_API_KEY;
if (!isValidPubloraKey(apiKey)) {
  throw new Error('Invalid PUBLORA_API_KEY format. Key must start with sk_');
}
```

```python
def is_valid_publora_key(key):
    """Validate Publora API key format."""
    if not key or not isinstance(key, str):
        return False
    if not key.startswith('sk_'):
        return False
    if len(key) < 20:
        return False
    return True

# Usage
api_key = os.environ.get('PUBLORA_API_KEY')
if not is_valid_publora_key(api_key):
    raise ValueError('Invalid PUBLORA_API_KEY format. Key must start with sk_')
```

## Two Ways to Authenticate

Publora provides two interfaces that use the **same API key** with **different header formats**:

| Interface | Header Format | Use Case |
|-----------|---------------|----------|
| REST API | `x-publora-key: sk_...` | Direct HTTP requests |
| MCP Server | `Authorization: Bearer sk_...` | AI assistants (Claude, Cursor) |

### REST API Authentication

For direct HTTP calls to `api.publora.com`:

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: sk_kzq5mjw.a1b2c3d4e5f6..."
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
        "Authorization": "Bearer sk_kzq5mjw.a1b2c3d4e5f6..."
      }
    }
  }
}
```

**Why different headers?** The REST API uses a custom header (`x-publora-key`) for simplicity. The MCP server uses the standard `Authorization: Bearer` header because MCP clients expect OAuth-style headers.

## Complete Authentication Workflow

### Step 1: Store Your Key Securely

```bash
# Add to ~/.bashrc or ~/.zshrc
export PUBLORA_API_KEY="sk_kzq5mjw.a1b2c3d4e5f6..."
```

Or use a `.env` file (add to `.gitignore`):

```bash
# .env
PUBLORA_API_KEY=sk_kzq5mjw.a1b2c3d4e5f6...
```

### Step 2: Verify Your Key Works

Test authentication by fetching your connected platforms:

```javascript
async function verifyAuthentication() {
  const apiKey = process.env.PUBLORA_API_KEY;

  // Validate format first
  if (!apiKey?.startsWith('sk_')) {
    console.error('Error: API key must start with sk_');
    console.error('Get your key at: https://publora.com → Settings → API Keys');
    return false;
  }

  try {
    const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
      headers: { 'x-publora-key': apiKey }
    });

    if (response.status === 401) {
      console.error('Error: Invalid API key');
      console.error('Your key may have been regenerated. Get a new one at publora.com');
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
        print('Get your key at: https://publora.com → Settings → API Keys')
        return False

    try:
        response = requests.get(
            'https://api.publora.com/api/v1/platform-connections',
            headers={'x-publora-key': api_key}
        )

        if response.status_code == 401:
            print('Error: Invalid API key')
            print('Your key may have been regenerated. Get a new one at publora.com')
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

For B2B workspaces managing multiple users, add the user ID header:

| Header | Value | Purpose |
|--------|-------|---------|
| `x-publora-key` | Your workspace API key | Authenticates your workspace |
| `x-publora-user-id` | Managed user's ID | Specifies which user to act as |

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': process.env.PUBLORA_WORKSPACE_KEY,
    'x-publora-user-id': '507f1f77bcf86cd799439011'  // Managed user
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
| 401 | `"Invalid API key"` | Key is wrong, missing, or revoked | Regenerate at Settings → API Keys |
| 403 | `"Subscription required"` | No active subscription | Subscribe at publora.com/pricing |
| 403 | `"Workspace access not enabled"` | Used `x-publora-user-id` without workspace | Enable workspace or remove header |

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
    throw new Error('Invalid API key. Regenerate at publora.com → Settings → API Keys');
  }

  if (response.status === 403) {
    const data = await response.json();
    if (data.error?.includes('Subscription')) {
      throw new Error('Subscription required. Subscribe at publora.com/pricing');
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
        raise PubloraAuthError('Invalid API key. Regenerate at publora.com → Settings → API Keys')

    if response.status_code == 403:
        data = response.json()
        if 'Subscription' in data.get('error', ''):
            raise PubloraAuthError('Subscription required. Subscribe at publora.com/pricing')
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
- Log full API keys (mask them: `sk_kzq...****`)

### Key Storage

```javascript
// Good: Environment variable
const apiKey = process.env.PUBLORA_API_KEY;

// Bad: Hardcoded
const apiKey = 'sk_kzq5mjw.a1b2c3d4e5f6...'; // Never do this!
```

### Key Regeneration

If your key is compromised:

1. Go to publora.com → Settings → API Keys
2. Click **Regenerate Key**
3. Update your environment variables
4. Old key is immediately invalidated

## Quick Reference

| What | Value |
|------|-------|
| REST API Base URL | `https://api.publora.com/api/v1` |
| MCP Server URL | `https://mcp.publora.com` |
| REST API Header | `x-publora-key: sk_...` |
| MCP Header | `Authorization: Bearer sk_...` |
| Key Format | `sk_` + timestamp + random hex |
| Key Expiration | Never (until regenerated) |
| Get Key | publora.com → Settings → API Keys |

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
