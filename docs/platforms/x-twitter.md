# X (Twitter)

Publora provides full integration with X (formerly Twitter), including text posts, media attachments, and automatic thread splitting for long-form content.

## Platform ID Format

```
twitter-{userId}
```

Where `{userId}` is your X/Twitter numeric user ID assigned during account connection.

## Requirements

- An X/Twitter account connected via OAuth through the Publora dashboard
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 280 characters (25,000 for Premium) |
| Images | Yes | Up to 4 per post, PNG preferred |
| Videos | Yes | MP4 format |
| Threads | Yes | Auto-split with `[1/N]` markers |

## Character Counting

X has specific rules for character counting that Publora handles automatically:

- Standard characters count as 1
- Emojis count as **2 characters**
- URLs are shortened to 23 characters regardless of length
- Publora calculates the true character count before posting

## Threading

When your content exceeds the character limit, Publora automatically splits it into a thread:

- Content is split at sentence boundaries when possible
- Each part is numbered with `[1/N]` markers (e.g., `[1/3]`, `[2/3]`, `[3/3]`)
- You can also manually define thread parts by separating them with `---`

## Examples

### Post a Text Update

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Hello from Publora! Posting to X has never been easier.',
    platforms: ['twitter-12345678']
  })
});

const data = await response.json();
console.log(data);
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
        'content': 'Hello from Publora! Posting to X has never been easier.',
        'platforms': ['twitter-12345678']
    }
)

data = response.json()
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Hello from Publora! Posting to X has never been easier.",
    "platforms": ["twitter-12345678"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Hello from Publora! Posting to X has never been easier.',
  platforms: ['twitter-12345678']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with an Image

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Check out this screenshot of our new dashboard!',
    platforms: ['twitter-12345678']
  })
});

const data = await response.json();
console.log(data);
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
        'content': 'Check out this screenshot of our new dashboard!',
        'platforms': ['twitter-12345678']
    }
)

data = response.json()
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Check out this screenshot of our new dashboard!",
    "platforms": ["twitter-12345678"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Check out this screenshot of our new dashboard!',
  platforms: ['twitter-12345678']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

> **Note:** To attach media to a post, first create the post, then use the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

### Post a Thread

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `Here is a deep dive into our new feature release and what it means for developers building on our platform.

We have completely redesigned the API layer to support batch operations, real-time webhooks, and granular rate limiting. This means you can now process up to 1000 requests per minute with predictable throughput.

The new SDK is available for JavaScript, Python, and Go. Each SDK includes full TypeScript definitions, async support, and built-in retry logic. Check our docs for migration guides.`,
    platforms: ['twitter-12345678']
  })
});

const data = await response.json();
console.log(data);
```

**Python (requests)**

```python
import requests

content = """Here is a deep dive into our new feature release and what it means for developers building on our platform.

We have completely redesigned the API layer to support batch operations, real-time webhooks, and granular rate limiting. This means you can now process up to 1000 requests per minute with predictable throughput.

The new SDK is available for JavaScript, Python, and Go. Each SDK includes full TypeScript definitions, async support, and built-in retry logic. Check our docs for migration guides."""

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': content,
        'platforms': ['twitter-12345678']
    }
)

data = response.json()
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Here is a deep dive into our new feature release and what it means for developers building on our platform.\n\nWe have completely redesigned the API layer to support batch operations, real-time webhooks, and granular rate limiting. This means you can now process up to 1000 requests per minute with predictable throughput.\n\nThe new SDK is available for JavaScript, Python, and Go. Each SDK includes full TypeScript definitions, async support, and built-in retry logic. Check our docs for migration guides.",
    "platforms": ["twitter-12345678"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const content = `Here is a deep dive into our new feature release and what it means for developers building on our platform.

We have completely redesigned the API layer to support batch operations, real-time webhooks, and granular rate limiting. This means you can now process up to 1000 requests per minute with predictable throughput.

The new SDK is available for JavaScript, Python, and Go. Each SDK includes full TypeScript definitions, async support, and built-in retry logic. Check our docs for migration guides.`;

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content,
  platforms: ['twitter-12345678']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

Publora will automatically split this into a numbered thread (e.g., `[1/3]`, `[2/3]`, `[3/3]`) at sentence boundaries.

## Platform Quirks

- **Emoji character counting**: Each emoji counts as 2 characters toward the 280-character limit. Publora accounts for this automatically.
- **PNG preferred for images**: While JPEG works, PNG images tend to render with higher quality on X due to their compression algorithm.
- **Thread numbering**: Publora adds `[1/N]` markers at the end of each tweet in a thread. This is appended after the content, so it reduces available character space by approximately 6-8 characters per tweet.
- **Premium character limit**: If your account has X Premium, the character limit increases to 25,000. Publora detects this based on your connected account.
- **Rate limits**: X enforces its own rate limits. If you hit them, Publora will return the appropriate error from the X API.
- **Up to 4 images**: You can attach a maximum of 4 images to a single tweet. Attempting to attach more will result in an error.
- **Video and images are mutually exclusive**: A single tweet can contain either images or a video, but not both.

## Character Limits

| Account Type | Limit |
|-------------|-------|
| Standard | 280 characters |
| X Premium | 25,000 characters |
| Thread tweet (each part) | Same as above, minus `[X/N]` marker space |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
