# Workspace / B2B API

This guide covers how to use the Publora Workspace API to manage multiple users under your account. This is designed for B2B integrations where you need to create and manage social media posting on behalf of your customers.

## How It Works

The Workspace API lets you create **managed users** under your workspace. Each managed user gets their own isolated environment with their own social media connections and posts. You control everything through your workspace API key.

### Core Concepts

- **Workspace:** Your B2B account that can manage multiple users.
- **Managed User:** A user created under your workspace. They have their own social connections and posts, but you control them via the API.
- **Connection URL:** A temporary OAuth link you generate and send to a managed user so they can connect their social media accounts.
- **Per-User API Key:** An optional API key generated for a specific managed user, allowing direct API access scoped to that user.

### Authentication Headers

| Header | Purpose |
|---|---|
| `x-publora-key` | Your workspace API key (required on all requests) |
| `x-publora-user-id` | The managed user's ID (required when acting on behalf of a user) |

### Rate Limits

Each managed user has a daily posting limit of **100 posts per day** (`dailyPostsLeft`).

### Endpoints Overview

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/v1/workspace/users` | List all managed users |
| `POST` | `/api/v1/workspace/users` | Create a new managed user |
| `DELETE` | `/api/v1/workspace/users/:userId` | Remove a managed user |
| `POST` | `/api/v1/workspace/users/:userId/api-key` | Generate a per-user API key |
| `POST` | `/api/v1/workspace/users/:userId/connection-url` | Generate an OAuth connection link |

## Examples

### Create a Managed User

**JavaScript (fetch)**

```javascript
const response = await fetch(
  'https://api.publora.com/api/v1/workspace/users',
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      email: 'client@example.com',
      displayName: 'Acme Corp'
    })
  }
);

const user = await response.json();
console.log('Managed user created:', user.id);
console.log('Email:', user.email);
console.log('Display name:', user.displayName);
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/workspace/users',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'email': 'client@example.com',
        'displayName': 'Acme Corp'
    }
)

user = response.json()
print(f"Managed user created: {user['id']}")
print(f"Email: {user['email']}")
print(f"Display name: {user['displayName']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/workspace/users \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "email": "client@example.com",
    "displayName": "Acme Corp"
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const { data: user } = await axios.post(
  'https://api.publora.com/api/v1/workspace/users',
  {
    email: 'client@example.com',
    displayName: 'Acme Corp'
  },
  {
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

console.log('Managed user created:', user.id);
console.log('Email:', user.email);
console.log('Display name:', user.displayName);
```

---

### List All Managed Users

**JavaScript (fetch)**

```javascript
const response = await fetch(
  'https://api.publora.com/api/v1/workspace/users',
  {
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

const users = await response.json();

console.log(`Total managed users: ${users.length}`);
for (const user of users) {
  console.log(`  - ${user.id}: ${user.displayName} (${user.email})`);
}
```

**Python (requests)**

```python
import requests

response = requests.get(
    'https://api.publora.com/api/v1/workspace/users',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)

users = response.json()

print(f"Total managed users: {len(users)}")
for user in users:
    print(f"  - {user['id']}: {user['displayName']} ({user['email']})")
```

**cURL**

```bash
curl -s https://api.publora.com/api/v1/workspace/users \
  -H "x-publora-key: YOUR_API_KEY" | jq '.[] | "\(.id): \(.displayName) (\(.email))"'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const { data: users } = await axios.get(
  'https://api.publora.com/api/v1/workspace/users',
  {
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

console.log(`Total managed users: ${users.length}`);
for (const user of users) {
  console.log(`  - ${user.id}: ${user.displayName} (${user.email})`);
}
```

---

### Generate a Connection URL

A connection URL is a temporary OAuth link that you send to your managed user. When they open it, they can authorize their social media accounts (Twitter, LinkedIn, Instagram, etc.) to be managed through your workspace.

**JavaScript (fetch)**

```javascript
const userId = 'user_abc123'; // The managed user's ID

const response = await fetch(
  `https://api.publora.com/api/v1/workspace/users/${userId}/connection-url`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

const data = await response.json();
console.log('Connection URL:', data.connectionUrl);
console.log('Expires:', data.expiresAt);

// Send this URL to your user via email, in-app notification, etc.
// When they open it, they will be guided through connecting their social accounts.
```

**Python (requests)**

```python
import requests

user_id = 'user_abc123'

response = requests.post(
    f'https://api.publora.com/api/v1/workspace/users/{user_id}/connection-url',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    }
)

data = response.json()
print(f"Connection URL: {data['connectionUrl']}")
print(f"Expires: {data['expiresAt']}")

# Send this URL to your user via email, in-app notification, etc.
```

**cURL**

```bash
curl -X POST "https://api.publora.com/api/v1/workspace/users/user_abc123/connection-url" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY"
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const userId = 'user_abc123';

const { data } = await axios.post(
  `https://api.publora.com/api/v1/workspace/users/${userId}/connection-url`,
  {},
  {
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

console.log('Connection URL:', data.connectionUrl);
console.log('Expires:', data.expiresAt);

// Send this URL to your user via email, in-app notification, etc.
```

---

### Generate a Per-User API Key

If you want a managed user to have their own API key (scoped only to their data), you can generate one.

**JavaScript (fetch)**

```javascript
const userId = 'user_abc123';

const response = await fetch(
  `https://api.publora.com/api/v1/workspace/users/${userId}/api-key`,
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

const data = await response.json();
console.log('Per-user API key:', data.apiKey);

// This key can be used in the x-publora-key header
// and will only have access to this specific user's data.
```

**Python (requests)**

```python
import requests

user_id = 'user_abc123'

response = requests.post(
    f'https://api.publora.com/api/v1/workspace/users/{user_id}/api-key',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    }
)

data = response.json()
print(f"Per-user API key: {data['apiKey']}")
```

**cURL**

```bash
curl -X POST "https://api.publora.com/api/v1/workspace/users/user_abc123/api-key" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY"
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const userId = 'user_abc123';

const { data } = await axios.post(
  `https://api.publora.com/api/v1/workspace/users/${userId}/api-key`,
  {},
  {
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

console.log('Per-user API key:', data.apiKey);
```

---

### Post on Behalf of a Managed User

To create posts for a managed user, include the `x-publora-user-id` header with the managed user's ID alongside your workspace API key.

**JavaScript (fetch)**

```javascript
const userId = 'user_abc123';

const response = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY',
    'x-publora-user-id': userId
  },
  body: JSON.stringify({
    text: 'Exciting update from Acme Corp! We just hit 10,000 customers.',
    platformIds: ['twitter-123456', 'linkedin-ABCDEF'],
    scheduledTime: '2026-03-15T14:00:00.000Z'
  })
});

const post = await response.json();
console.log(`Post created for user ${userId}:`, post.id);
```

**Python (requests)**

```python
import requests

user_id = 'user_abc123'

response = requests.post(
    'https://api.publora.com/api/v1/post-groups',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY',
        'x-publora-user-id': user_id
    },
    json={
        'text': 'Exciting update from Acme Corp! We just hit 10,000 customers.',
        'platformIds': ['twitter-123456', 'linkedin-ABCDEF'],
        'scheduledTime': '2026-03-15T14:00:00.000Z'
    }
)

post = response.json()
print(f"Post created for user {user_id}: {post['id']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "x-publora-user-id: user_abc123" \
  -d '{
    "text": "Exciting update from Acme Corp! We just hit 10,000 customers.",
    "platformIds": ["twitter-123456", "linkedin-ABCDEF"],
    "scheduledTime": "2026-03-15T14:00:00.000Z"
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const userId = 'user_abc123';

const { data: post } = await axios.post(
  'https://api.publora.com/api/v1/post-groups',
  {
    text: 'Exciting update from Acme Corp! We just hit 10,000 customers.',
    platformIds: ['twitter-123456', 'linkedin-ABCDEF'],
    scheduledTime: '2026-03-15T14:00:00.000Z'
  },
  {
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY',
      'x-publora-user-id': userId
    }
  }
);

console.log(`Post created for user ${userId}:`, post.id);
```

---

### Delete a Managed User

**JavaScript (fetch)**

```javascript
const userId = 'user_abc123';

const response = await fetch(
  `https://api.publora.com/api/v1/workspace/users/${userId}`,
  {
    method: 'DELETE',
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

if (response.ok) {
  console.log(`User ${userId} has been removed.`);
} else {
  const error = await response.json();
  console.error('Failed to delete user:', error.error || error.message);
}
```

**Python (requests)**

```python
import requests

user_id = 'user_abc123'

response = requests.delete(
    f'https://api.publora.com/api/v1/workspace/users/{user_id}',
    headers={'x-publora-key': 'YOUR_API_KEY'}
)

if response.ok:
    print(f"User {user_id} has been removed.")
else:
    error = response.json()
    print(f"Failed to delete user: {error.get('error') or error.get('message')}")
```

**cURL**

```bash
curl -X DELETE "https://api.publora.com/api/v1/workspace/users/user_abc123" \
  -H "x-publora-key: YOUR_API_KEY"
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const userId = 'user_abc123';

try {
  await axios.delete(
    `https://api.publora.com/api/v1/workspace/users/${userId}`,
    {
      headers: { 'x-publora-key': 'YOUR_API_KEY' }
    }
  );

  console.log(`User ${userId} has been removed.`);
} catch (error) {
  console.error('Failed to delete user:', error.response?.data?.error);
}
```

---

### Full B2B Onboarding Workflow

This example shows the complete workflow: create a managed user, generate a connection URL, wait for them to connect, then post on their behalf.

**JavaScript (fetch)**

```javascript
async function onboardClient(email, displayName) {
  const headers = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  };

  // Step 1: Create the managed user
  const createResponse = await fetch(
    'https://api.publora.com/api/v1/workspace/users',
    {
      method: 'POST',
      headers,
      body: JSON.stringify({ email, displayName })
    }
  );

  const user = await createResponse.json();
  console.log(`1. Created user: ${user.id} (${user.displayName})`);

  // Step 2: Generate a connection URL for them to auth their social accounts
  const connectionResponse = await fetch(
    `https://api.publora.com/api/v1/workspace/users/${user.id}/connection-url`,
    {
      method: 'POST',
      headers
    }
  );

  const connectionData = await connectionResponse.json();
  console.log(`2. Connection URL: ${connectionData.connectionUrl}`);
  console.log(`   Expires: ${connectionData.expiresAt}`);
  console.log('   Send this link to the client so they can connect their social accounts.');

  // Step 3: (After user connects accounts) Check their connections
  // In production, you would wait for a webhook or poll periodically.
  const connectionsResponse = await fetch(
    'https://api.publora.com/api/v1/connections',
    {
      headers: {
        'x-publora-key': 'YOUR_API_KEY',
        'x-publora-user-id': user.id
      }
    }
  );

  const connections = await connectionsResponse.json();
  console.log(`3. User has ${connections.length} connected accounts.`);

  if (connections.length === 0) {
    console.log('   User has not connected any accounts yet.');
    return user;
  }

  // Step 4: Post on their behalf
  const platformIds = connections.map(c => c.platformId);

  const postResponse = await fetch('https://api.publora.com/api/v1/post-groups', {
    method: 'POST',
    headers: {
      ...headers,
      'x-publora-user-id': user.id
    },
    body: JSON.stringify({
      text: `Welcome to ${displayName}'s social presence, powered by our platform!`,
      platformIds,
      scheduledTime: '2026-03-15T14:00:00.000Z'
    })
  });

  const post = await postResponse.json();
  console.log(`4. Post scheduled for user ${user.id}: ${post.id}`);

  return user;
}

// Usage
const client = await onboardClient('client@example.com', 'Acme Corp');
```

**Python (requests)**

```python
import requests


def onboard_client(email, display_name):
    api_url = 'https://api.publora.com/api/v1'
    headers = {
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    }

    # Step 1: Create the managed user
    user_response = requests.post(
        f'{api_url}/workspace/users',
        headers=headers,
        json={'email': email, 'displayName': display_name}
    )
    user = user_response.json()
    print(f"1. Created user: {user['id']} ({user['displayName']})")

    # Step 2: Generate a connection URL
    connection_response = requests.post(
        f"{api_url}/workspace/users/{user['id']}/connection-url",
        headers=headers
    )
    connection_data = connection_response.json()
    print(f"2. Connection URL: {connection_data['connectionUrl']}")
    print(f"   Expires: {connection_data['expiresAt']}")
    print('   Send this link to the client.')

    # Step 3: Check their connections (after they connect)
    user_headers = {**headers, 'x-publora-user-id': user['id']}
    connections_response = requests.get(
        f'{api_url}/connections',
        headers=user_headers
    )
    connections = connections_response.json()
    print(f"3. User has {len(connections)} connected accounts.")

    if not connections:
        print('   User has not connected any accounts yet.')
        return user

    # Step 4: Post on their behalf
    platform_ids = [c['platformId'] for c in connections]
    post_response = requests.post(
        f'{api_url}/post-groups',
        headers=user_headers,
        json={
            'text': f"Welcome to {display_name}'s social presence, powered by our platform!",
            'platformIds': platform_ids,
            'scheduledTime': '2026-03-15T14:00:00.000Z'
        }
    )
    post = post_response.json()
    print(f"4. Post scheduled for user {user['id']}: {post['id']}")

    return user


# Usage
client = onboard_client('client@example.com', 'Acme Corp')
```

## Best Practices

1. **Store managed user IDs.** After creating a managed user, persist their `id` in your own database. You will need it for all subsequent operations on their behalf.

2. **Send connection URLs promptly.** Connection URLs expire after a set number of days. Send them to your users as soon as they are generated, and regenerate if needed.

3. **Use per-user API keys for client-side integrations.** If your managed users need to interact with the API directly (e.g., from their own dashboard), generate per-user API keys instead of sharing your workspace key.

4. **Never expose your workspace API key.** Your workspace key has full access to all managed users. Keep it server-side only. Use per-user API keys for any client-facing scenarios.

5. **Monitor `dailyPostsLeft`.** Each managed user is limited to 100 posts per day. If you are building a high-volume integration, track this limit and queue posts accordingly.

6. **Clean up unused users.** If a managed user is no longer needed (e.g., they cancel their subscription with you), delete them via the API to keep your workspace clean.

7. **Always include both headers when acting on behalf of a user.** For any post, connection, or media operation on a managed user's behalf, you need both `x-publora-key` (your workspace key) and `x-publora-user-id` (the managed user's ID).

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| `401` when using workspace endpoints | Invalid workspace API key | Verify the `x-publora-key` value is your workspace-level API key |
| `404` when posting on behalf of a user | Missing or invalid `x-publora-user-id` header | Ensure the header is present and contains a valid managed user ID |
| User has no connections | They have not opened the connection URL yet, or the URL expired | Generate a new connection URL and send it to the user |
| Connection URL expired | The URL has a limited lifespan | Generate a fresh connection URL via `POST /workspace/users/:userId/connection-url` |
| `403` daily limit reached | Managed user has used all 100 daily posts | Wait until the next day or contact Publora to increase the limit |
| Posts not appearing for a managed user | Using your own API key without the `x-publora-user-id` header | Include the `x-publora-user-id` header so the post is created under the managed user's account |
| Per-user API key does not work | Key was regenerated, invalidating the old one | Use the latest generated key; generating a new key invalidates previous keys |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
