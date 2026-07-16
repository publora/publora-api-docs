# Mastodon API - Post to Fediverse via REST API

Post to Mastodon and the Fediverse programmatically using the Publora REST API. A simpler alternative to the official Mastodon API, Megalodon, or direct ActivityPub integration.

## Mastodon API Overview

Publora provides a unified REST API for publishing text posts and media content to Mastodon and the fediverse. Posts are published to the mastodon.social instance. No need to manage Mastodon OAuth flows, handle ActivityPub protocols, or set up your own Mastodon application.

### Why Use Publora Instead of Mastodon API / Megalodon?

| Feature | Publora API | Mastodon API |
|---------|-------------|--------------|
| Authentication | Single API key | OAuth 2.0 per instance |
| Instance support | mastodon.social | Any instance |
| Multi-platform | Post to 10 platforms | Mastodon/Fediverse only |
| Setup time | 5 minutes | Varies by instance |
| Media handling | Automatic | Manual upload |
| Federation | Automatic | Automatic |

### Keywords: Mastodon API, Mastodon posting API, Fediverse API, ActivityPub API, post to Mastodon programmatically, Mastodon REST API, Mastodon developer API, Mastodon bot API, Mastodon automation API, toot API, Mastodon status API, decentralized social API

## Platform ID Format

```
mastodon-{accountId}
```

Where `{accountId}` is your Mastodon account ID assigned during account connection through the Publora dashboard.

## Requirements

- A Mastodon account on the mastodon.social instance connected through the Publora dashboard (OAuth scopes requested: `read`, `write`, `push`)
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 500 characters |
| Images | Yes | JPEG, PNG, GIF, WebP, HEIF, HEIC, AVIF; up to 4 per post |
| Videos | Yes | MP4, WebM, MOV formats |
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
        'content': 'Hello fediverse! We just shipped a major update to our open-source project. Check out the changelog at https://example.com/changelog #opensource #fediverse',
        'platforms': ['mastodon-109876543210']
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
    "content": "Hello fediverse! We just shipped a major update to our open-source project. Check out the changelog at https://example.com/changelog #opensource #fediverse",
    "platforms": ["mastodon-109876543210"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
    platforms: ['mastodon-109876543210']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

> **Note:** To attach media to a Mastodon post, first create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

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
        'platforms': ['mastodon-109876543210']
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
    "content": "New feature alert: our dashboard now supports dark mode! Here is a side-by-side comparison. #ui #darkmode",
    "platforms": ["mastodon-109876543210"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'New feature alert: our dashboard now supports dark mode! Here is a side-by-side comparison. #ui #darkmode',
  platforms: ['mastodon-109876543210']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
    platforms: ['mastodon-109876543210']
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
        'content': 'Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!',
        'platforms': ['mastodon-109876543210']
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
    "content": "Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!",
    "platforms": ["mastodon-109876543210"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Quick demo of our new real-time collaboration feature. Multiple users editing the same document simultaneously!',
  platforms: ['mastodon-109876543210']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

## Platform Quirks

- **mastodon.social only**: New connections are limited to the mastodon.social instance (the OAuth flow uses a hardcoded instance URL). The test-connection validator attempts to extract the instance URL from the connection's `profileUrl` field, but `profileUrl` is never set during Mastodon connection creation, so it always falls back to mastodon.social. Support for other instances may be added in the future.
- **Public by default**: All posts made through Publora are published with public visibility. They will appear on the federated timeline.
- **Up to 4 images**: A maximum of 4 images can be attached to a single post. Publora enforces this limit at scheduling time via `postValidationService.js` and will reject posts that exceed it before they reach the Mastodon API.
- **Image formats**: Publora's validator accepts JPEG, PNG, GIF, WebP, HEIF, HEIC, and AVIF for Mastodon. The publisher passes supported media through rather than converting it.
- **MP4, WebM, and MOV for videos**: Mastodon accepts MP4, WebM, and MOV video formats. Publora accepts all three as input, but the scheduler currently reports the MIME type as `video/mp4` to Mastodon regardless of the actual format. MP4 uploads work correctly; WebM and MOV files may experience processing issues due to the mismatched MIME type.
- **500-character limit**: Mastodon enforces a strict 500-character limit. Publora will return an error if your content exceeds this. Mastodon and Meta Threads do not auto-thread in Publora; only X/Twitter threading is currently enabled.
- **Hashtags**: Hashtags in Mastodon are part of the post body and count toward the character limit. They become clickable and searchable on the platform.
- **Content warnings**: Mastodon supports content warnings (CW), but this feature is not currently available through the Publora API.
- **Federation delay**: Because Mastodon is federated, posts may take a few seconds to propagate to other instances in the fediverse.

## API Limits

### Text Limits

| Element | Limit |
|---------|-------|
| Post body | 500 characters (instance-configurable, some allow 5,000+) |
| Media description (alt text) | 1,500 characters |

### Media Limits

| Media Type | Max Size | Max Count | Supported Formats |
|------------|----------|-----------|-------------------|
| Images | 16 MB | 4 per post | JPEG, PNG, GIF, WebP, HEIF, HEIC, AVIF |
| Videos | ~99 MB | 1 per post | MP4, WebM, MOV |

| Video Constraint | Limit |
|------------------|-------|
| Duration | 86,400 seconds (24 hours) |

### Rate Limits

| Limit Type | Value |
|------------|-------|
| Media uploads | 30 per 30 minutes |
| API requests | 300 per 5 minutes |

### Additional Notes

- Character limits vary by Mastodon instance; mastodon.social uses 500 characters, but some instances allow 5,000+
- Publora currently connects to mastodon.social only (posting is hardcoded to this instance)
- Mastodon and Meta Threads do not support auto-threading in the current Publora capability set; X/Twitter threading remains enabled
- Max image count (4) and video count (1) limits are enforced by Publora at scheduling time via `postValidationService.js`

## What you can't do through the REST API

- **Set media descriptions (alt text):** The Mastodon publisher forwards `file.description` when present, but the API media model has no persisted `description` field. The current REST upload flows therefore cannot supply it.

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
