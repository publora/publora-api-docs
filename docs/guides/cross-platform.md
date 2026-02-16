# Cross-Platform Posting

This guide covers how to publish a single post to multiple social media platforms simultaneously through the Publora API, including platform-specific behavior and content adaptation.

## How It Works

Publora lets you compose a single post and distribute it across multiple social media platforms in one API call. The API handles platform-specific formatting, character limits, and media requirements automatically.

### Platform IDs

Each connected social account is identified by a **platform ID** in the format:

```
{platform}-{platformId}
```

For example:
- `twitter-123456` -- a connected Twitter/X account
- `linkedin-ABCDEF` -- a connected LinkedIn profile
- `instagram-789012` -- a connected Instagram account
- `tiktok-345678` -- a connected TikTok account
- `facebook-901234` -- a connected Facebook page
- `youtube-567890` -- a connected YouTube channel
- `threads-111213` -- a connected Threads account
- `telegram-141516` -- a connected Telegram channel
- `bluesky-171819` -- a connected Bluesky account
- `pinterest-202122` -- a connected Pinterest account

Retrieve your connected accounts and their platform IDs from the `GET /api/v1/connections` endpoint.

### Character Limits by Platform

| Platform | Character Limit |
|---|---|
| Twitter / X | 280 |
| LinkedIn | 3,000 |
| Threads | 500 |
| Telegram | 1,024 (text) / 4,096 (with media caption) |
| Facebook | 63,206 |
| Instagram | 2,200 |
| TikTok | 2,200 |
| YouTube | 5,000 (description) |
| Bluesky | 300 |
| Pinterest | 500 |

### Automatic Content Adaptation

When your content exceeds a platform's character limit, Publora adapts it automatically:

- **Twitter / X:** Long text is split into a **thread** (multiple tweets chained together).
- **Threads:** Long text is split into a **thread** (multiple posts chained together).
- **Other platforms:** Content is truncated to fit the platform's limit, with an effort to break at natural word boundaries.

### Platform-Specific Defaults

Publora applies sensible defaults for platform-specific settings:

- **TikTok:** Default privacy and interaction settings are applied automatically.
- **Instagram:** Video posts are published as **Reels** by default.
- **YouTube:** Videos are set to **public** visibility by default.

## Examples

### Post to All Connected Platforms

**JavaScript (fetch)**

```javascript
// First, get all connected platform IDs
const connectionsResponse = await fetch(
  'https://api.publora.com/api/v1/connections',
  {
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

const connections = await connectionsResponse.json();
const platformIds = connections.map(c => c.platformId);

console.log('Connected platforms:', platformIds);

// Create a post targeting all platforms
const postResponse = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    text: 'We just launched our new feature! Check it out at https://example.com',
    platformIds: platformIds,
    scheduledTime: '2026-03-15T14:00:00.000Z'
  })
});

const post = await postResponse.json();
console.log('Post scheduled to all platforms:', post.id);
```

**Python (requests)**

```python
import requests

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

# Get all connected platform IDs
connections_response = requests.get(
    f'{API_URL}/connections',
    headers=HEADERS
)

connections = connections_response.json()
platform_ids = [c['platformId'] for c in connections]
print(f"Connected platforms: {platform_ids}")

# Create a post targeting all platforms
post_response = requests.post(
    f'{API_URL}/post-groups',
    headers=HEADERS,
    json={
        'text': 'We just launched our new feature! Check it out at https://example.com',
        'platformIds': platform_ids,
        'scheduledTime': '2026-03-15T14:00:00.000Z'
    }
)

post = post_response.json()
print(f"Post scheduled to all platforms: {post['id']}")
```

**cURL**

```bash
# Get all connected platform IDs
CONNECTIONS=$(curl -s https://api.publora.com/api/v1/connections \
  -H "x-publora-key: YOUR_API_KEY")

echo "Connected platforms:"
echo "$CONNECTIONS" | jq '.[].platformId'

# Extract platform IDs into a JSON array
PLATFORM_IDS=$(echo "$CONNECTIONS" | jq '[.[].platformId]')

# Create a post targeting all platforms
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d "{
    \"text\": \"We just launched our new feature! Check it out at https://example.com\",
    \"platformIds\": $PLATFORM_IDS,
    \"scheduledTime\": \"2026-03-15T14:00:00.000Z\"
  }"
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

// Get all connected platform IDs
const { data: connections } = await api.get('/connections');
const platformIds = connections.map(c => c.platformId);

console.log('Connected platforms:', platformIds);

// Create a post targeting all platforms
const { data: post } = await api.post('/post-groups', {
  text: 'We just launched our new feature! Check it out at https://example.com',
  platformIds: platformIds,
  scheduledTime: '2026-03-15T14:00:00.000Z'
});

console.log('Post scheduled to all platforms:', post.id);
```

---

### Post to Specific Platforms (Twitter + LinkedIn + Instagram)

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    text: 'Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.',
    platformIds: [
      'twitter-123456',
      'linkedin-ABCDEF',
      'instagram-789012'
    ],
    scheduledTime: '2026-03-15T09:00:00.000Z'
  })
});

const post = await response.json();
console.log('Post created:', post.id);
// On Twitter, if the text exceeds 280 characters, it becomes a thread automatically.
// On LinkedIn (3000 char limit) and Instagram (2200 char limit), the full text is posted as-is.
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/post-groups',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'text': 'Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.',
        'platformIds': [
            'twitter-123456',
            'linkedin-ABCDEF',
            'instagram-789012'
        ],
        'scheduledTime': '2026-03-15T09:00:00.000Z'
    }
)

post = response.json()
print(f"Post created: {post['id']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "text": "Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.",
    "platformIds": [
      "twitter-123456",
      "linkedin-ABCDEF",
      "instagram-789012"
    ],
    "scheduledTime": "2026-03-15T09:00:00.000Z"
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const { data: post } = await axios.post(
  'https://api.publora.com/api/v1/post-groups',
  {
    text: 'Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.',
    platformIds: [
      'twitter-123456',
      'linkedin-ABCDEF',
      'instagram-789012'
    ],
    scheduledTime: '2026-03-15T09:00:00.000Z'
  },
  {
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

console.log('Post created:', post.id);
```

---

### Handle Long Content Across Platforms

When posting long-form content, Publora automatically adapts it for each platform. Here is an example with text that exceeds Twitter's 280-character limit.

**JavaScript (fetch)**

```javascript
const longText = `We are thrilled to announce that our platform now supports 10 social media channels!

Here is what's new:
- Twitter/X threading for long posts
- LinkedIn articles with rich formatting
- Instagram Reels for video content
- TikTok with automatic settings
- YouTube with public visibility defaults
- Facebook pages
- Threads support
- Telegram channels
- Bluesky integration
- Pinterest pins

This has been months in the making, and we cannot wait for you to try it out. Visit our website to learn more and connect your accounts today.`;

const response = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    text: longText,
    platformIds: [
      'twitter-123456',   // Will become a thread (exceeds 280 chars)
      'linkedin-ABCDEF',  // Full text posted (under 3000 chars)
      'threads-111213',    // Will become a thread (exceeds 500 chars)
      'telegram-141516',   // Handled within Telegram limits
      'bluesky-171819'     // Adapted for 300 char limit
    ],
    scheduledTime: '2026-03-15T10:00:00.000Z'
  })
});

const post = await response.json();
console.log('Post created:', post.id);

// After publishing, check individual platform results
const statusResponse = await fetch(
  `https://api.publora.com/api/v1/post-groups/${post.id}`,
  {
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

const status = await statusResponse.json();
console.log('Overall status:', status.status);

// Each platform post may have its own status
if (status.posts) {
  for (const platformPost of status.posts) {
    console.log(`${platformPost.platform}: ${platformPost.status}`);
  }
}
```

**Python (requests)**

```python
import requests

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

long_text = """We are thrilled to announce that our platform now supports 10 social media channels!

Here is what's new:
- Twitter/X threading for long posts
- LinkedIn articles with rich formatting
- Instagram Reels for video content
- TikTok with automatic settings
- YouTube with public visibility defaults
- Facebook pages
- Threads support
- Telegram channels
- Bluesky integration
- Pinterest pins

This has been months in the making, and we cannot wait for you to try it out. Visit our website to learn more and connect your accounts today."""

response = requests.post(
    f'{API_URL}/post-groups',
    headers=HEADERS,
    json={
        'text': long_text,
        'platformIds': [
            'twitter-123456',
            'linkedin-ABCDEF',
            'threads-111213',
            'telegram-141516',
            'bluesky-171819'
        ],
        'scheduledTime': '2026-03-15T10:00:00.000Z'
    }
)

post = response.json()
print(f"Post created: {post['id']}")

# Check individual platform results after publishing
status_response = requests.get(
    f"{API_URL}/post-groups/{post['id']}",
    headers=HEADERS
)

status = status_response.json()
print(f"Overall status: {status['status']}")

if 'posts' in status:
    for platform_post in status['posts']:
        print(f"  {platform_post['platform']}: {platform_post['status']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "text": "We are thrilled to announce that our platform now supports 10 social media channels!\n\nHere is what'\''s new:\n- Twitter/X threading for long posts\n- LinkedIn articles with rich formatting\n- Instagram Reels for video content\n- TikTok with automatic settings\n- YouTube with public visibility defaults\n- Facebook pages\n- Threads support\n- Telegram channels\n- Bluesky integration\n- Pinterest pins\n\nThis has been months in the making, and we cannot wait for you to try it out.",
    "platformIds": [
      "twitter-123456",
      "linkedin-ABCDEF",
      "threads-111213",
      "telegram-141516",
      "bluesky-171819"
    ],
    "scheduledTime": "2026-03-15T10:00:00.000Z"
  }'

# Check post status (replace POST_GROUP_ID with the actual ID)
curl -s https://api.publora.com/api/v1/post-groups/POST_GROUP_ID \
  -H "x-publora-key: YOUR_API_KEY" | jq '.status, .posts[]?.platform, .posts[]?.status'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

const longText = `We are thrilled to announce that our platform now supports 10 social media channels!

Here is what's new:
- Twitter/X threading for long posts
- LinkedIn articles with rich formatting
- Instagram Reels for video content
- TikTok with automatic settings
- YouTube with public visibility defaults
- Facebook pages
- Threads support
- Telegram channels
- Bluesky integration
- Pinterest pins

This has been months in the making, and we cannot wait for you to try it out. Visit our website to learn more and connect your accounts today.`;

const { data: post } = await api.post('/post-groups', {
  text: longText,
  platformIds: [
    'twitter-123456',
    'linkedin-ABCDEF',
    'threads-111213',
    'telegram-141516',
    'bluesky-171819'
  ],
  scheduledTime: '2026-03-15T10:00:00.000Z'
});

console.log('Post created:', post.id);

// Check platform-level results after publishing
const { data: status } = await api.get(`/post-groups/${post.id}`);
console.log('Overall status:', status.status);

if (status.posts) {
  for (const platformPost of status.posts) {
    console.log(`  ${platformPost.platform}: ${platformPost.status}`);
  }
}
```

## Best Practices

1. **Keep text concise for multi-platform posts.** If you want the same text to appear identically on all platforms, keep it under 280 characters (the smallest common limit for Twitter/X). Otherwise, expect automatic threading or truncation.

2. **Retrieve platform IDs dynamically.** Do not hardcode platform IDs. Use `GET /api/v1/connections` to discover connected accounts, since users may add or remove connections.

3. **Check post status after publishing.** A post group may end up `partially_published` if some platforms succeed and others fail. Always inspect individual platform post statuses.

4. **Be aware of media requirements per platform.** Instagram requires media on every post. TikTok requires video. If you include platforms with different media requirements, ensure your media satisfies all of them, or split into separate post groups.

5. **Test with a single platform first.** When developing your integration, start by posting to one platform, verify it works, and then expand to multiple platforms.

6. **Use the same post group for analytics.** Since all platform posts belong to one post group, you can track the overall performance of a campaign through a single ID.

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| Post succeeds on some platforms but not others | Different requirements per platform (e.g., Instagram needs media) | Check which platforms need media and ensure your post includes it, or create separate post groups |
| Twitter post appears as a thread | Text exceeds 280 characters | This is expected behavior -- Publora auto-threads long Twitter content |
| `400` error with invalid platform ID | Platform ID does not match the `{platform}-{id}` format, or account is not connected | Verify the format and check `GET /api/v1/connections` for valid IDs |
| Content looks truncated on some platforms | Platform character limit is lower than your text length | Review the character limits table above and shorten content if exact wording matters |
| Video post fails on Instagram | Instagram requires specific video formats for Reels | Ensure your video is MP4, meets Instagram's aspect ratio requirements, and is within duration limits |
