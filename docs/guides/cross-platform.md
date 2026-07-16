# Cross-Platform Posting

This guide covers how to publish a single post to multiple social media platforms simultaneously through the Publora API, including platform-specific behavior and content adaptation.

## How It Works

Publora lets you compose a single post and distribute it across multiple social media platforms in one API call. The API handles platform-specific formatting, character limits, and media requirements automatically.

### Plan Restrictions

> **API access note:** API access is controlled by the `apiAccess` entitlement on your account. This entitlement is **enabled by default on all plans, including the free Starter plan** (15 posts/month, 3 accounts); only custom accounts with `apiAccess` explicitly disabled are blocked. If your account does not have the `apiAccess` entitlement, API calls will return a `403` error. The same applies to `mcpAccess` for MCP integrations.

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
- `mastodon-202122` -- a connected Mastodon account

Retrieve your connected accounts and their platform IDs from the `GET /api/v1/platform-connections` endpoint.

> **Pinterest:** Pinterest is registered in the platform registry (`SUPPORTED_PLATFORM_KEYS`) but publishing support is not yet fully available or tested. Pinterest support is on the roadmap.

### Character Limits by Platform

| Platform | Character Limit |
|---|---|
| Twitter / X | 280 |
| LinkedIn | 3,000 |
| Threads | 500 |
| Telegram | 4,096 (text) / 1,024 (media captions) |
| Facebook | 63,206 |
| Instagram | 2,200 |
| TikTok | 2,200 |
| YouTube | 5,000 (description) |
| Bluesky | 300 |
| Mastodon | 500 |

### Automatic Content Adaptation

When your content exceeds a platform's character limit, Publora adapts it automatically:

- **Twitter / X:** Long text is split into a **thread** (multiple tweets chained together).
- **Threads:** Long text is split into a **thread** (multiple posts chained together). ⚠️ *Temporarily unavailable - see note below.*
- **Other platforms:** Content that exceeds the platform's character limit will return a validation error. Content is **not** auto-truncated for non-threading platforms.

> **⚠️ Threads Notice:** Multi-part thread splitting on Threads is temporarily unavailable due to API access requirements. Keep Threads content under 500 characters or it will fail. Single posts and carousels work normally. Contact support@publora.com for updates.

### Platform-Specific Defaults

`platformSettings` can be passed directly in the `create-post` request body. The API merges user-provided settings with defaults per platform.
>
> **⚠️ Supported platforms for `platformSettings`:** The API merges `platformSettings` for **TikTok**, **Instagram**, **YouTube**, **Threads**, **Telegram**, and **LinkedIn**. Unknown top-level platforms or nested fields are rejected with `400 PLATFORM_SETTING_UNKNOWN`; they are not silently ignored. See the canonical [create-post platformSettings allowlist](../endpoints/create-post.md#unknown-platformsettings-paths).
>
> **⚠️ Important limitation:** The external API `update-post` endpoint accepts `status`, `scheduledTime`, `platformSettings` (merged per-platform), and `mediaUrls`. `mediaUrls` has **append semantics**: newly ingested media is added to existing media rather than replacing it, and ingestion is limited to 60 URLs/hour. It does **not** accept `content` or `platforms`. See [update-post](../endpoints/update-post.md). To change the text or target platforms after creation, use the Publora dashboard or create a new post.

Publora applies sensible defaults for platform-specific settings:

- **TikTok:** Default privacy and interaction settings are applied automatically.
- **Instagram:** Video posts are published as **Reels** by default. Set `videoType: "STORIES"` to post as a Story instead. An optional `coverUrl` (public JPEG URL) sets a custom Reels cover.
- **YouTube:** Videos are set to **public** visibility by default.
- **LinkedIn:** Repost settings can specify the parent post URN and `PUBLIC` or `CONNECTIONS` visibility. `CONNECTIONS` is personal-profile-only; company-page reposts must use `PUBLIC` and otherwise return `400`.

### Response Status Code

The `create-post` endpoint returns HTTP `200` (not `201`) on success. This applies to all post creation calls, including scheduled posts. Check `response.ok` or `response.status === 200` in your code.

## Examples

### Post to All Connected Platforms

**JavaScript (fetch)**

```javascript
// First, get all connected platform IDs
const connectionsResponse = await fetch(
  'https://api.publora.com/api/v1/platform-connections',
  {
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

const { connections } = await connectionsResponse.json();
const platforms = connections.map(c => c.platformId);

console.log('Connected platforms:', platforms);

// Create a post targeting all platforms
const postResponse = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'We just launched our new feature! Check it out at https://example.com',
    platforms: platforms,
    scheduledTime: '2026-03-15T14:00:00.000Z'
  })
});

const post = await postResponse.json();
console.log('Post scheduled to all platforms:', post.postGroupId);
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
    f'{API_URL}/platform-connections',
    headers=HEADERS
)

connections = connections_response.json()['connections']
platforms = [c['platformId'] for c in connections]
print(f"Connected platforms: {platforms}")

# Create a post targeting all platforms
post_response = requests.post(
    f'{API_URL}/create-post',
    headers=HEADERS,
    json={
        'content': 'We just launched our new feature! Check it out at https://example.com',
        'platforms': platforms,
        'scheduledTime': '2026-03-15T14:00:00.000Z'
    }
)

post = post_response.json()
print(f"Post scheduled to all platforms: {post['postGroupId']}")
```

**cURL**

```bash
# Get all connected platform IDs
CONNECTIONS=$(curl -s https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY")

echo "Connected platforms:"
echo "$CONNECTIONS" | jq '.connections[].platformId'

# Extract platform IDs into a JSON array
PLATFORMS=$(echo "$CONNECTIONS" | jq '[.connections[].platformId]')

# Create a post targeting all platforms
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d "{
    \"content\": \"We just launched our new feature! Check it out at https://example.com\",
    \"platforms\": $PLATFORMS,
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
const { data: connectionsData } = await api.get('/platform-connections');
const platforms = connectionsData.connections.map(c => c.platformId);

console.log('Connected platforms:', platforms);

// Create a post targeting all platforms
const { data: post } = await api.post('/create-post', {
  content: 'We just launched our new feature! Check it out at https://example.com',
  platforms: platforms,
  scheduledTime: '2026-03-15T14:00:00.000Z'
});

console.log('Post scheduled to all platforms:', post.postGroupId);
```

---

### Post to Specific Platforms (Twitter + LinkedIn + Instagram)

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.',
    platforms: [
      'twitter-123456',
      'linkedin-ABCDEF',
      'instagram-789012'
    ],
    scheduledTime: '2026-03-15T09:00:00.000Z'
  })
});

const post = await response.json();
console.log('Post created:', post.postGroupId);
// On Twitter, text above the connected account's applicable limit becomes a thread automatically.
// On LinkedIn (3000 char limit) and Instagram (2200 char limit), the full text is posted as-is.
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.',
        'platforms': [
            'twitter-123456',
            'linkedin-ABCDEF',
            'instagram-789012'
        ],
        'scheduledTime': '2026-03-15T09:00:00.000Z'
    }
)

post = response.json()
print(f"Post created: {post['postGroupId']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.",
    "platforms": [
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
  'https://api.publora.com/api/v1/create-post',
  {
    content: 'Big news! We are expanding to 3 new markets this quarter. Stay tuned for more details.',
    platforms: [
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

console.log('Post created:', post.postGroupId);
```

---

### Handle Long Content Across Platforms

When posting long-form content, Publora automatically adapts it for each platform. Here is an example with text that exceeds a standard Twitter account's 280-character limit.

**JavaScript (fetch)**

```javascript
const longContent = `We are thrilled to announce that our platform now supports 10 social media channels!

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
- Mastodon support

This has been months in the making, and we cannot wait for you to try it out. Visit our website to learn more and connect your accounts today.`;

const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: longContent,
    platforms: [
      'twitter-123456',   // Becomes a thread on a standard account (exceeds 280 chars)
      'linkedin-ABCDEF',  // Full text posted (under 3000 chars)
      'threads-111213',    // Will become a thread (exceeds 500 chars)
      'telegram-141516',   // Handled within Telegram limits
      'bluesky-171819'     // Bluesky: 300 char limit (no threading — content must fit within limit)
    ],
    scheduledTime: '2026-03-15T10:00:00.000Z'
  })
});

const post = await response.json();
console.log('Post created:', post.postGroupId);

// After publishing, check individual platform results
const statusResponse = await fetch(
  `https://api.publora.com/api/v1/get-post/${post.postGroupId}`,
  {
    headers: { 'x-publora-key': 'YOUR_API_KEY' }
  }
);

const status = await statusResponse.json();

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

long_content = """We are thrilled to announce that our platform now supports 10 social media channels!

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
- Mastodon support

This has been months in the making, and we cannot wait for you to try it out. Visit our website to learn more and connect your accounts today."""

response = requests.post(
    f'{API_URL}/create-post',
    headers=HEADERS,
    json={
        'content': long_content,
        'platforms': [
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
print(f"Post created: {post['postGroupId']}")

# Check individual platform results after publishing
status_response = requests.get(
    f"{API_URL}/get-post/{post['postGroupId']}",
    headers=HEADERS
)

status = status_response.json()

if 'posts' in status:
    for platform_post in status['posts']:
        print(f"  {platform_post['platform']}: {platform_post['status']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "We are thrilled to announce that our platform now supports 10 social media channels!\n\nHere is what'\''s new:\n- Twitter/X threading for long posts\n- LinkedIn articles with rich formatting\n- Instagram Reels for video content\n- TikTok with automatic settings\n- YouTube with public visibility defaults\n- Facebook pages\n- Threads support\n- Telegram channels\n- Bluesky integration\n- Mastodon support\n\nThis has been months in the making, and we cannot wait for you to try it out.",
    "platforms": [
      "twitter-123456",
      "linkedin-ABCDEF",
      "threads-111213",
      "telegram-141516",
      "bluesky-171819"
    ],
    "scheduledTime": "2026-03-15T10:00:00.000Z"
  }'

# Check post status (replace POST_GROUP_ID with the actual postGroupId)
curl -s https://api.publora.com/api/v1/get-post/POST_GROUP_ID \
  -H "x-publora-key: YOUR_API_KEY" | jq '.posts[]? | {platform, status}'
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

const longContent = `We are thrilled to announce that our platform now supports 10 social media channels!

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
- Mastodon support

This has been months in the making, and we cannot wait for you to try it out. Visit our website to learn more and connect your accounts today.`;

const { data: post } = await api.post('/create-post', {
  content: longContent,
  platforms: [
    'twitter-123456',
    'linkedin-ABCDEF',
    'threads-111213',
    'telegram-141516',
    'bluesky-171819'
  ],
  scheduledTime: '2026-03-15T10:00:00.000Z'
});

console.log('Post created:', post.postGroupId);

// Check platform-level results after publishing
const { data: status } = await api.get(`/get-post/${post.postGroupId}`);

if (status.posts) {
  for (const platformPost of status.posts) {
    console.log(`  ${platformPost.platform}: ${platformPost.status}`);
  }
}
```

## Best Practices

1. **Keep text concise for multi-platform posts.** If you want the same text to appear identically on all platforms, keep it under 280 characters (the smallest common limit for Twitter/X). Otherwise, expect automatic threading on supported platforms or a validation error on platforms that do not support threading.

2. **Retrieve platform IDs dynamically.** Do not hardcode platform IDs. Use `GET /api/v1/platform-connections` to discover connected accounts, since users may add or remove connections.

3. **Check post status after publishing.** A post group may end up `partially_published` if some platforms succeed and others fail. Always inspect individual platform post statuses via `GET /api/v1/get-post/:postGroupId`.

4. **Be aware of media requirements per platform.** Instagram requires media; TikTok accepts a photo carousel or one video; YouTube requires video. Ensure shared media satisfies every target or use separate post groups.

5. **Test with a single platform first.** When developing your integration, start by posting to one platform, verify it works, and then expand to multiple platforms.

6. **Use the same post group for analytics.** Since all platform posts belong to one post group, you can track the overall performance of a campaign through a single postGroupId.

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| Post succeeds on some platforms but not others | Different requirements per platform (e.g., Instagram needs media) | Check which platforms need media and ensure your post includes it, or create separate post groups |
| Twitter post appears as a thread | Text exceeds the connected account's applicable limit | This is expected behavior -- Publora auto-threads long Twitter content |
| `400` error with invalid platform ID | Platform ID does not match the `{platform}-{id}` format, or account is not connected | Verify the format and check `GET /api/v1/platform-connections` for valid IDs |
| Content rejected on some platforms | Platform character limit is lower than your text length and platform does not support threading | Shorten content to fit within the platform's limit, or post to those platforms separately with shorter text |
| Video post fails on Instagram | Instagram requires specific video formats for Reels | Ensure your video is MP4, meets Instagram's aspect ratio requirements, and is within duration limits |
| `403` `PLATFORM_NOT_AVAILABLE` | The target platform connection is not available on the account (e.g. the account is not connected) | Connect the account in the dashboard, then retry |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
