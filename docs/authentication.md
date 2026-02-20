# Authentication

## API Key Authentication

All API requests require the `x-publora-key` header. Publora uses API keys (not OAuth tokens) for authentication - no token refresh is needed.

### Getting Your API Key

1. Sign in at [publora.com](https://publora.com)
2. Go to **Settings > API Keys**
3. Click **Generate API Key**
4. Copy immediately -- the key is shown only once

**Important:** Your API key grants full access to your account. Treat it like a password.

### Key Format

```
sk_kzq5mjw.a1b2c3d4e5f6...
```

Keys start with `sk_` followed by a timestamp and a random hex string.

### Using Your Key

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: {
    'x-publora-key': 'sk_kzq5mjw.a1b2c3d4e5f6...'
  }
});
```

#### Python (requests)

```python
import requests

response = requests.get(
    'https://api.publora.com/api/v1/platform-connections',
    headers={'x-publora-key': 'sk_kzq5mjw.a1b2c3d4e5f6...'}
)
```

#### cURL

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: sk_kzq5mjw.a1b2c3d4e5f6..."
```

#### Node.js (axios)

```javascript
const axios = require('axios');

const client = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: { 'x-publora-key': 'sk_kzq5mjw.a1b2c3d4e5f6...' }
});

const { data } = await client.get('/platform-connections');
```

### Environment Variable Best Practice

Never hardcode API keys. Use environment variables:

```javascript
// JavaScript
const API_KEY = process.env.PUBLORA_API_KEY;

const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: { 'x-publora-key': API_KEY }
});
```

```python
# Python
import os
import requests

API_KEY = os.environ['PUBLORA_API_KEY']

response = requests.get(
    'https://api.publora.com/api/v1/platform-connections',
    headers={'x-publora-key': API_KEY}
)
```

```bash
# Bash
export PUBLORA_API_KEY="sk_kzq5mjw.a1b2c3d4e5f6..."

curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: $PUBLORA_API_KEY"
```

## Workspace / B2B Authentication

For workspace users managing multiple accounts, add the user ID header:

```
x-publora-key: YOUR_WORKSPACE_API_KEY
x-publora-user-id: MANAGED_USER_ID
```

This lets workspace owners create posts and manage media on behalf of managed users.

### JavaScript

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_WORKSPACE_API_KEY',
    'x-publora-user-id': '507f1f77bcf86cd799439011'
  },
  body: JSON.stringify({
    content: 'Post on behalf of a managed user',
    platforms: ['twitter-123456']
  })
});
```

### Python

```python
response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_WORKSPACE_API_KEY',
        'x-publora-user-id': '507f1f77bcf86cd799439011'
    },
    json={
        'content': 'Post on behalf of a managed user',
        'platforms': ['twitter-123456']
    }
)
```

## Error Responses

| Status | Error | Meaning |
|--------|-------|---------|
| 401 | `"Invalid API key"` | Key is missing, malformed, or revoked |
| 403 | `"Subscription required"` | No active subscription on this account |
| 403 | `"Workspace access is not enabled"` | `x-publora-user-id` used but workspace not enabled |

## Key Security

- Keys are bcrypt-hashed in the database -- we never store plaintext
- Generate a new key anytime via Settings
- Old keys are immediately invalidated when you generate a new one
- Never share keys in public repos, chat messages, or client-side code

## Error Handling and Retry Logic

### JavaScript with Retry

```javascript
async function publoraRequest(endpoint, options = {}, maxRetries = 3) {
  const baseUrl = 'https://api.publora.com/api/v1';
  const apiKey = process.env.PUBLORA_API_KEY;

  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(`${baseUrl}${endpoint}`, {
        ...options,
        headers: {
          'x-publora-key': apiKey,
          'Content-Type': 'application/json',
          ...options.headers
        }
      });

      if (response.status === 401) {
        throw new Error('Invalid API key. Check your PUBLORA_API_KEY environment variable.');
      }

      if (response.status === 429) {
        // Rate limited - wait and retry
        const retryAfter = response.headers.get('Retry-After') || 60;
        console.log(`Rate limited. Waiting ${retryAfter}s before retry...`);
        await new Promise(r => setTimeout(r, retryAfter * 1000));
        continue;
      }

      if (response.status >= 500 && attempt < maxRetries) {
        // Server error - exponential backoff
        const delay = Math.pow(2, attempt) * 1000;
        console.log(`Server error. Retrying in ${delay}ms...`);
        await new Promise(r => setTimeout(r, delay));
        continue;
      }

      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || `HTTP ${response.status}`);
      }

      return data;
    } catch (error) {
      if (attempt === maxRetries) throw error;
    }
  }
}

// Usage
const connections = await publoraRequest('/platform-connections');
const post = await publoraRequest('/create-post', {
  method: 'POST',
  body: JSON.stringify({ content: 'Hello!', platforms: ['twitter-123'] })
});
```

### Python with Retry

```python
import os
import time
import requests
from functools import wraps

class PubloraClient:
    def __init__(self, api_key=None):
        self.api_key = api_key or os.environ.get('PUBLORA_API_KEY')
        if not self.api_key:
            raise ValueError('PUBLORA_API_KEY environment variable not set')
        self.base_url = 'https://api.publora.com/api/v1'
        self.session = requests.Session()
        self.session.headers.update({
            'x-publora-key': self.api_key,
            'Content-Type': 'application/json'
        })

    def _request(self, method, endpoint, max_retries=3, **kwargs):
        url = f'{self.base_url}{endpoint}'

        for attempt in range(1, max_retries + 1):
            try:
                response = self.session.request(method, url, **kwargs)

                if response.status_code == 401:
                    raise ValueError('Invalid API key')

                if response.status_code == 429:
                    retry_after = int(response.headers.get('Retry-After', 60))
                    print(f'Rate limited. Waiting {retry_after}s...')
                    time.sleep(retry_after)
                    continue

                if response.status_code >= 500 and attempt < max_retries:
                    delay = 2 ** attempt
                    print(f'Server error. Retrying in {delay}s...')
                    time.sleep(delay)
                    continue

                response.raise_for_status()
                return response.json()

            except requests.RequestException as e:
                if attempt == max_retries:
                    raise

        return None

    def get_connections(self):
        return self._request('GET', '/platform-connections')

    def create_post(self, content, platforms, scheduled_time=None):
        payload = {'content': content, 'platforms': platforms}
        if scheduled_time:
            payload['scheduledTime'] = scheduled_time
        return self._request('POST', '/create-post', json=payload)

    def get_post(self, post_group_id):
        return self._request('GET', f'/get-post/{post_group_id}')

# Usage
client = PubloraClient()
connections = client.get_connections()
post = client.create_post('Hello!', ['twitter-123'], '2026-03-01T10:00:00Z')
```

## Complete Authentication Workflow

Here's a complete example showing how to authenticate and make your first API call:

### Step 1: Set Up Environment

```bash
# Store your API key securely
export PUBLORA_API_KEY="sk_your_key_here"
```

### Step 2: Verify Authentication

```javascript
// Verify your API key is valid by fetching connections
async function verifyApiKey() {
  try {
    const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
      headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
    });

    if (response.status === 401) {
      console.error('Invalid API key. Please check your key at publora.com/settings');
      return false;
    }

    const data = await response.json();
    console.log(`Authenticated! ${data.connections.length} connected accounts.`);
    return true;
  } catch (error) {
    console.error('Connection error:', error.message);
    return false;
  }
}
```

```python
def verify_api_key():
    """Verify API key is valid."""
    response = requests.get(
        'https://api.publora.com/api/v1/platform-connections',
        headers={'x-publora-key': os.environ['PUBLORA_API_KEY']}
    )

    if response.status_code == 401:
        print('Invalid API key. Check your key at publora.com/settings')
        return False

    data = response.json()
    print(f"Authenticated! {len(data['connections'])} connected accounts.")
    return True
```

### Step 3: Handle Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `401 Invalid API key` | Key is wrong, expired, or missing | Regenerate key at Settings > API Keys |
| `403 Subscription required` | No active subscription | Subscribe at publora.com/pricing |
| `429 Too Many Requests` | Rate limit exceeded | Wait and retry with exponential backoff |
| `500 Internal Server Error` | Server issue | Retry after a few seconds |

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
