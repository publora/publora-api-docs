# JavaScript Quick Start

Get started with the Publora API in JavaScript. These examples use the native `fetch` API.

## Setup

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

const headers = {
  'Content-Type': 'application/json',
  'x-publora-key': PUBLORA_API_KEY
};
```

## List Connected Accounts

```javascript
async function getConnections() {
  const response = await fetch(`${BASE_URL}/platform-connections`, {
    headers: { 'x-publora-key': PUBLORA_API_KEY }
  });

  const data = await response.json();
  console.log('Connected accounts:', data.connections);
  return data.connections;
}

// Example output:
// [
//   { platformId: 'twitter-123456', username: '@myaccount', displayName: 'My Account' },
//   { platformId: 'linkedin-ABC123', username: 'johndoe', displayName: 'John Doe' }
// ]
```

## Create a Simple Post

```javascript
async function createPost(content, platforms) {
  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      content,
      platforms
    })
  });

  const data = await response.json();
  console.log('Post created:', data.postGroupId);
  return data;
}

// Usage
await createPost(
  'Hello from Publora API!',
  ['twitter-123456', 'linkedin-ABC123']
);
```

## Schedule a Post for Later

```javascript
async function schedulePost(content, platforms, scheduledTime) {
  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      content,
      platforms,
      scheduledTime // ISO 8601 UTC format
    })
  });

  return response.json();
}

// Schedule for tomorrow at 2 PM UTC
const tomorrow = new Date();
tomorrow.setDate(tomorrow.getDate() + 1);
tomorrow.setUTCHours(14, 0, 0, 0);

await schedulePost(
  'Scheduled post going live tomorrow!',
  ['twitter-123456'],
  tomorrow.toISOString()
);
```

## Check Post Status

```javascript
async function getPostStatus(postGroupId) {
  const response = await fetch(`${BASE_URL}/get-post/${postGroupId}`, {
    headers: { 'x-publora-key': PUBLORA_API_KEY }
  });

  const data = await response.json();

  // Check status of each platform
  data.posts.forEach(post => {
    console.log(`${post.platform}: ${post.status}`);
  });

  return data;
}

// Statuses: 'scheduled', 'published', 'failed', 'draft'
```

## Delete a Post

```javascript
async function deletePost(postGroupId) {
  const response = await fetch(`${BASE_URL}/delete-post/${postGroupId}`, {
    method: 'DELETE',
    headers: { 'x-publora-key': PUBLORA_API_KEY }
  });

  if (response.ok) {
    console.log('Post deleted successfully');
  }

  return response.json();
}
```

## Complete Example

```javascript
// Full workflow: list accounts, create post, check status
async function main() {
  // 1. Get connected accounts
  const connections = await getConnections();

  if (connections.length === 0) {
    console.log('No accounts connected. Visit app.publora.com to connect.');
    return;
  }

  // 2. Get platform IDs
  const platforms = connections
    .map(c => c.platformId)
    .filter(id => !/^(instagram|tiktok|youtube|pinterest)-/.test(id));

  if (platforms.length === 0) {
    console.log('No text-capable connections found. Attach media via mediaUrls or use the media upload flow.');
    return;
  }
  console.log('Posting to:', platforms);

  // 3. Schedule a text post five minutes from now
  const result = await schedulePost(
    'Testing the Publora API - works great!',
    platforms,
    new Date(Date.now() + 5 * 60 * 1000).toISOString()
  );

  if (!result.success) {
    throw new Error(result.message || result.error || 'Failed to schedule post');
  }

  // 4. Check status after a moment
  setTimeout(async () => {
    await getPostStatus(result.postGroupId);
  }, 5000);
}

main().catch(console.error);
```

---

*[Publora](https://publora.com) — Social media API with free tier, paid plans from $2.99/account*
