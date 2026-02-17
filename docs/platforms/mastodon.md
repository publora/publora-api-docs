# Mastodon

Publora integrates with Mastodon for publishing text posts and media content to the fediverse. Posts are published to the mastodon.social instance.

## Platform ID Format

```
mastodon-{accountId}
```

Where `{accountId}` is your Mastodon account ID assigned during account connection through the Publora dashboard.

## Requirements

- A Mastodon account on the mastodon.social instance connected through the Publora dashboard
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 500 characters |
| Images | Yes | JPEG, PNG, up to 4 per post |
| Videos | Yes | MP4 format |
| Visibility | Yes | Public by default |

## Visibility

Posts on Mastodon are set to **public** visibility by default when posted through Publora. This means they will appear on your profile, in your followers' home timelines, and on the public federated timeline.

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
    content: 'Hello fediverse! We just shipped a major update to our open-source project. Check out the changelog at https://example.com/changelog #opensource #fediverse',
    platforms: ['mastodon-109876543210']
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
        'content': 'Hello fediverse! We just shipped a major update to our open-source project. Check out the changelog at https://example.com/changelog #opensource #fediverse',
        'platforms': ['mastodon-109876543210']
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
    "content": "Hello fediverse! We just shipped a major update to our open-source project. Check out the changelog at https://example.com/changelog #opensource #fediverse",
    "platforms": ["mastodon-109876543210"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Hello fediverse! We just shipped a major update to our open-source project. Check out the changelog at https://example.com/changelog #opensource #fediverse',
  platforms: ['mastodon-109876543210']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with Media

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'New feature alert: our dashboard now supports dark mode! Here is a side-by-side comparison. #ui #darkmode',
    platforms: ['mastodon-109876543210'],
    mediaUrls: [
      'https://example.com/images/light-mode.png',
      'https://example.com/images/dark-mode.png'
    ]
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
        'content': 'New feature alert: our dashboard now supports dark mode! Here is a side-by-side comparison. #ui #darkmode',
        'platforms': ['mastodon-109876543210'],
        'mediaUrls': [
            'https://example.com/images/light-mode.png',
            'https://example.com/images/dark-mode.png'
        ]
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
    "content": "New feature alert: our dashboard now supports dark mode! Here is a side-by-side comparison. #ui #darkmode",
    "platforms": ["mastodon-109876543210"],
    "mediaUrls": [
      "https://example.com/images/light-mode.png",
      "https://example.com/images/dark-mode.png"
    ]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'New feature alert: our dashboard now supports dark mode! Here is a side-by-side comparison. #ui #darkmode',
  platforms: ['mastodon-109876543210'],
  mediaUrls: [
    'https://example.com/images/light-mode.png',
    'https://example.com/images/dark-mode.png'
  ]
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with a Video

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!',
    platforms: ['mastodon-109876543210'],
    mediaUrls: ['https://example.com/videos/collab-demo.mp4']
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
        'content': 'Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!',
        'platforms': ['mastodon-109876543210'],
        'mediaUrls': ['https://example.com/videos/collab-demo.mp4']
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
    "content": "Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!",
    "platforms": ["mastodon-109876543210"],
    "mediaUrls": ["https://example.com/videos/collab-demo.mp4"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!',
  platforms: ['mastodon-109876543210'],
  mediaUrls: ['https://example.com/videos/collab-demo.mp4']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

## Platform Quirks

- **mastodon.social instance**: Publora currently connects to the mastodon.social instance. Support for other instances may be added in the future.
- **Public by default**: All posts made through Publora are published with public visibility. They will appear on the federated timeline.
- **Up to 4 images**: A maximum of 4 images can be attached to a single post. Exceeding this limit will result in an error.
- **JPEG and PNG only**: Mastodon supports JPEG and PNG image formats. Other formats may not be accepted.
- **MP4 for videos**: Only MP4 video format is supported for media attachments.
- **500-character limit**: Mastodon enforces a strict 500-character limit. Publora will return an error if your content exceeds this. Unlike X/Twitter and Threads, Mastodon does not auto-thread.
- **Hashtags**: Hashtags in Mastodon are part of the post body and count toward the character limit. They become clickable and searchable on the platform.
- **Content warnings**: Mastodon supports content warnings (CW), but this feature is not currently available through the Publora API.
- **Federation delay**: Because Mastodon is federated, posts may take a few seconds to propagate to other instances in the fediverse.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 500 characters |
| Images | Up to 4 per post |
| Media description (alt text) | 1,500 characters |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
