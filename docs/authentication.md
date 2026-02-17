# Authentication

## API Key Authentication

All API requests require the `x-publora-key` header.

### Getting Your API Key

1. Sign in at [publora.com](https://publora.com)
2. Go to **Settings > API Keys**
3. Click **Generate API Key**
4. Copy immediately -- the key is shown only once

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


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
