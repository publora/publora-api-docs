# Twitter API / X API - Post Tweets via REST API

Post to Twitter (X) programmatically using the Publora REST API. A simpler alternative to the official Twitter API v2, Tweepy, node-twitter-api-v2, or Twitter Java SDK.

## Twitter API Overview

Publora provides a unified REST API for posting tweets to X (formerly Twitter), including text posts, media attachments, images, videos, and automatic thread splitting for long-form content. No need to manage OAuth tokens, API rate limits, or complex Twitter API authentication flows.

### Why Use Publora Instead of Twitter API v2 / Tweepy / node-twitter-api-v2?

| Feature | Publora API | Twitter API v2 / Tweepy |
|---------|-------------|-------------------------|
| Authentication | Single API key | Complex OAuth 2.0 flow |
| Rate limit handling | Automatic | Manual implementation |
| Thread creation | Automatic splitting | Manual tweet chaining |
| Multi-platform | Post to 10 platforms | Twitter only |
| Setup time | 5 minutes | Hours to days |
| Pricing | API access is included on the free Starter plan | Free tier + paid tiers |

### Keywords: Twitter API, X API, post tweet API, Twitter posting API, tweet programmatically, Twitter bot API, Twitter automation API, send tweet API, Twitter REST API, Twitter developer API

## Platform ID Format

```
twitter-{userId}
```

Where `{userId}` is your X/Twitter numeric user ID assigned during account connection.

## Requirements

- An X/Twitter account connected via OAuth through the Publora dashboard
- API key from Publora
- Publora API access on any standard plan, including the free Starter plan

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 280 characters (standard) / 25,000 characters (Premium and PremiumPlus) |
| Images | Yes | Up to 4 per post, auto-converted to PNG (max 1000px width) |
| Videos | Yes | MP4, MOV format |
| Threads | Yes | Auto-split with `(1/N)` markers |

## Character Counting

X has specific rules for character counting that Publora handles automatically:

- Standard characters count as 1
- Emojis count as **2 characters**
- URLs are counted by their literal length (Publora does NOT apply Twitter's 23-character URL shortening rule)
- Publora calculates the character count before posting

## Threading

When your content exceeds the character limit, Publora automatically splits it into a thread (multiple connected tweets):

### How It Works

Publora uses the official X API v2 `reply.in_reply_to_tweet_id` parameter to chain tweets together. Each subsequent tweet is posted as a reply to the previous one, creating a connected thread.

**Technical flow:**
1. First tweet is published normally via `POST /2/tweets`
2. Each subsequent tweet is published with `reply.in_reply_to_tweet_id` set to the previous tweet's ID
3. All tweets share the same `conversation_id` (equals the first tweet's ID)

### Automatic Splitting

When content exceeds the applicable account limit, Publora automatically:

- Splits at paragraph breaks (`\n\n`) when possible
- Falls back to sentence boundaries (`. `, `! `, `? `)
- Falls back to word boundaries if needed
- Adds `(1/N)` markers at the end of each tweet (e.g., `(1/3)`, `(2/3)`, `(3/3)`)
- Reserves 10 characters per tweet for the marker

> **Note:** Publora auto-converts all images to PNG and resizes them to a maximum of 1000px width using sharp before uploading.

### Manual Thread Parts

You can manually define where thread breaks should occur using either method:

**Method 1: Triple dash separator**
```
This is my first tweet in the thread.

---

This is my second tweet in the thread.

---

And this is my third tweet!
```

**Method 2: Explicit markers**
```
First part of the thread [1/3]

Second part of the thread [2/3]

Third and final part [3/3]
```

When explicit markers are detected, Publora preserves them exactly as written and splits at those points. Use square brackets `[n/m]`, which is distinct from the auto-added numbering format `(1/N)` that uses parentheses.

### Media in Threads

- **Images:** Up to 4 images attached to the first tweet only
- **Video:** Single video attached to the first tweet only
- Subsequent tweets in the thread are text-only
- Images and video cannot be combined in the same tweet

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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

Publora will automatically split this into a numbered thread (e.g., `(1/3)`, `(2/3)`, `(3/3)`) at sentence boundaries.

## Platform Quirks

- **Emoji character counting**: Each emoji counts as 2 characters toward the applicable account limit. Publora accounts for this automatically.
- **PNG preferred for images**: While JPEG works, PNG images tend to render with higher quality on X due to their compression algorithm.
- **Thread numbering**: Publora adds `(1/N)` markers at the end of each tweet in a thread. This is appended after the content, so it reduces available character space by 10 characters per tweet.
- **Image auto-conversion**: Publora automatically converts all images to PNG format and resizes them to a maximum width of 1000px before uploading.
- **Rate limits**: X enforces its own rate limits. If you hit them, Publora will return the appropriate error from the X API.
- **Up to 4 images**: You can attach a maximum of 4 images to a single tweet. Attempting to attach more will result in an error.
- **Video and images are mutually exclusive**: A single tweet can contain either images or a video, but not both.

## API Limits

<!-- limits tables below synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->

### Character Limit

- **Standard accounts:** 280 characters
- **Premium and PremiumPlus accounts:** 25,000 characters
- **Detection:** Publora selects the limit automatically from the connected X account's subscription type
- **Threading:** Supported when content exceeds the applicable account limit

### Image Limits

| Property | Limit |
|----------|-------|
| Max size | 5 MB |
| Max count | 4 per tweet |
| Formats | JPEG, PNG, GIF, WebP |

> **Note:** All images are automatically converted to PNG (max 1000px width) via sharp before upload, regardless of the original format. This means animated GIFs will lose their animation.

### Video Limits (API)

| Property | Limit |
|----------|-------|
| **Max duration** | **2 minutes 20 seconds (140 seconds)** |
| Max size | 512 MB |
| Formats | MP4, MOV |

> **Important:** Publora validates X videos at **140 seconds (2:20)** through the API.

**Premium users via native app:** Up to 4 hours (not available via API)

## Character Limits

| Account Type | Limit |
|-------------|-------|
| Standard | 280 characters |
| Premium and PremiumPlus | 25,000 characters |
| Thread tweet (each part) | Applicable account limit, minus `(X/N)` marker space (10 chars) |

## Rate Limits

X-side pricing and posting quotas change independently of Publora and are not a Publora contract. Consult X's current developer documentation; Publora surfaces platform rate-limit errors when X rejects a request.

## What you can't do

- **Preserve animated GIFs:** The X publish path sends every non-video media item through `sharp(...).png()` before upload. A GIF is accepted as input, but animation is lost and X receives a static PNG.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
