# Threads

Publora integrates with Threads (by Meta) for publishing text posts, media, carousels, and automatic thread splitting for long-form content.

## Platform ID Format

```
threads-{accountId}
```

Where `{accountId}` is your Threads account ID assigned during connection via Meta OAuth.

## Requirements

- A Threads account connected via Meta OAuth through the Publora dashboard
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 500 characters |
| Images | Yes | Carousel supported, WebP auto-converted |
| Videos | Yes | MP4 format |
| Threads | Yes | Auto-split for long content |
| Hashtags | Yes | Maximum 1 hashtag per post |

## Threading

When your content exceeds the 500-character limit, Publora automatically splits it into a thread:

- Content is split at sentence boundaries when possible
- Each part respects the 500-character limit
- You can also manually define thread parts by separating them with `---`
- Unlike X/Twitter, Threads does not use `[1/N]` markers by default, but you can include them manually

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
    content: 'Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.',
    platforms: ['threads-55667788']
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
        'content': 'Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.',
        'platforms': ['threads-55667788']
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
    "content": "Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.",
    "platforms": ["threads-55667788"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.',
  platforms: ['threads-55667788']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with an Image Carousel

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Our product evolution over the past year. #buildinpublic',
    platforms: ['threads-55667788']
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
        'content': 'Our product evolution over the past year. #buildinpublic',
        'platforms': ['threads-55667788']
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
    "content": "Our product evolution over the past year. #buildinpublic",
    "platforms": ["threads-55667788"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Our product evolution over the past year. #buildinpublic',
  platforms: ['threads-55667788']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

> **Note:** To attach media to a Threads post, first create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

### Post a Thread (Long Content)

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.

First, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.

The actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step.`,
    platforms: ['threads-55667788']
  })
});

const data = await response.json();
console.log(data);
```

**Python (requests)**

```python
import requests

content = """We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.

First, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.

The actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step."""

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': content,
        'platforms': ['threads-55667788']
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
    "content": "We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.\n\nFirst, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.\n\nThe actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step.",
    "platforms": ["threads-55667788"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const content = `We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.

First, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.

The actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step.`;

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content,
  platforms: ['threads-55667788']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

Publora will automatically split this into multiple thread posts, each staying within the 500-character limit.

## Platform Quirks

- **Single hashtag limit**: Threads allows a maximum of 1 hashtag per post. If your content includes more than one hashtag, only the first will be recognized by the platform.
- **WebP auto-conversion**: If you provide WebP images, Publora automatically converts them before uploading to Threads.
- **Auto-threading**: Content exceeding 500 characters is automatically split into a thread. Publora splits at sentence boundaries to keep posts readable.
- **Manual thread parts**: You can use `---` as a separator in your content to explicitly define where thread breaks should occur.
- **No edit support**: Once posted, Threads posts cannot be edited via the API. You would need to delete and repost.
- **MP4 for videos**: Only MP4 video format is supported. Other formats will be rejected.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 500 characters |
| Hashtags | 1 per post |
| Thread parts | No fixed limit on number of parts |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
