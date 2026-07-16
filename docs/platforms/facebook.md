# Facebook API - Post to Facebook Page via REST API

Post to Facebook Pages programmatically using the Publora REST API. A simpler alternative to the Facebook Graph API or Meta Marketing API.

## Facebook API Overview

Publora provides a unified REST API for publishing text posts, images (including carousels and albums), and video content to Facebook Pages. Multiple pages per account are supported. No need to manage Facebook OAuth flows, handle Graph API versioning, or set up a Meta Developer app.

### Why Use Publora Instead of Facebook Graph API?

| Feature | Publora API | Facebook Graph API |
|---------|-------------|-------------------|
| Authentication | Single API key | OAuth 2.0 flow |
| API versioning | Handled | Manual updates required |
| Multi-page support | Built-in | Manual implementation |
| Multi-platform | Post to 10 platforms | Facebook only |
| Setup time | 5 minutes | Hours (app setup) |
| Media handling | Automatic | Manual upload |

### Keywords: Facebook API, Facebook Graph API, Facebook posting API, Facebook Page API, post to Facebook programmatically, Facebook REST API, Facebook developer API, Facebook automation API, Facebook content API, Facebook bot API, Meta Graph API

## Platform ID Format

```
facebook-{pageId}
```

Where `{pageId}` is your Facebook Page ID assigned during account connection via Facebook OAuth. If you manage multiple pages, each page gets its own platform ID.

## Requirements

- A Facebook Page (not a personal profile) connected via OAuth through the Publora dashboard
- Page admin permissions granted during OAuth (OAuth scopes requested: `public_profile`, `pages_manage_posts`, `pages_show_list`, `pages_read_engagement`, `business_management`)
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 2,200 characters (Publora frontend limit; see note below) |
| Images | Yes | Multiple supported (carousel/album), WebP auto-converted to JPEG |
| Videos | Yes | MP4 and MOV formats |
| Multiple Pages | Yes | Each page has its own platform ID |

## Token Management

Facebook page tokens have a **59-day lifespan**. Publora automatically handles token refresh so you do not need to manually reconnect your account. However, if the refresh fails (e.g., due to permission changes on Facebook), you will need to reconnect the page through the Publora dashboard.

> **Note:** The auto-refresh mechanism is handled by a separate internal service and runs independently of the API.

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
    content: 'We are thrilled to announce our new product launch! After months of development, we are finally ready to share what we have been building. Visit our website to learn more and sign up for early access.',
    platforms: ['facebook-112233445566']
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
        'content': 'We are thrilled to announce our new product launch! After months of development, we are finally ready to share what we have been building. Visit our website to learn more and sign up for early access.',
        'platforms': ['facebook-112233445566']
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
    "content": "We are thrilled to announce our new product launch! After months of development, we are finally ready to share what we have been building. Visit our website to learn more and sign up for early access.",
    "platforms": ["facebook-112233445566"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'We are thrilled to announce our new product launch! After months of development, we are finally ready to share what we have been building. Visit our website to learn more and sign up for early access.',
  platforms: ['facebook-112233445566']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

### Post with Multiple Images

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Highlights from our company retreat last weekend! Great team, great memories.',
    platforms: ['facebook-112233445566']
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
        'content': 'Highlights from our company retreat last weekend! Great team, great memories.',
        'platforms': ['facebook-112233445566']
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
    "content": "Highlights from our company retreat last weekend! Great team, great memories.",
    "platforms": ["facebook-112233445566"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Highlights from our company retreat last weekend! Great team, great memories.',
  platforms: ['facebook-112233445566']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

> **Note:** To attach media to a Facebook post, first create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

### Post to Multiple Facebook Pages

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Important announcement: Our offices will be closed on Monday for the holiday. We will resume normal business hours on Tuesday.',
    platforms: [
      'facebook-112233445566',
      'facebook-778899001122'
    ]
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
        'content': 'Important announcement: Our offices will be closed on Monday for the holiday. We will resume normal business hours on Tuesday.',
        'platforms': [
            'facebook-112233445566',
            'facebook-778899001122'
        ]
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
    "content": "Important announcement: Our offices will be closed on Monday for the holiday. We will resume normal business hours on Tuesday.",
    "platforms": [
      "facebook-112233445566",
      "facebook-778899001122"
    ]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Important announcement: Our offices will be closed on Monday for the holiday. We will resume normal business hours on Tuesday.',
  platforms: [
    'facebook-112233445566',
    'facebook-778899001122'
  ]
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

- **Pages only, not profiles**: Publora posts to Facebook Pages, not personal profiles. The Facebook API does not allow posting to personal profiles via third-party apps.
- **Multiple pages**: If you manage multiple Facebook Pages, each page is connected separately and gets its own `facebook-{pageId}` platform ID. You can post to multiple pages in a single API call.
- **WebP auto-conversion**: WebP images are automatically converted to JPEG before uploading to Facebook. No action needed on your part.
- **59-day token lifespan**: Facebook page access tokens expire after 59 days. Publora auto-refreshes these tokens, but if the refresh fails silently (e.g., due to permission changes on Facebook's side), posts will start failing without a specific token-related error. If you experience unexplained posting failures, try reconnecting the page through the Publora dashboard.
- **Album behavior**: When posting multiple images, Facebook may display them as a carousel or an album depending on the count and the viewer's device.
- **Video and images are separate**: A single post can contain either images or a video, but not both simultaneously. This is a Facebook platform limitation. Note that Publora's backend validation does **not** enforce this mixed media restriction for Facebook (it only does so for Instagram). If Facebook rejects mixed media, the error comes from the Facebook API itself, not from Publora's validation layer.
- **Multi-media code path supports images only**: The multi-media code path currently only supports multiple images (photo albums). Multiple videos in a single post are not properly supported — if you attach multiple videos, they will be processed through the photo upload path incorrectly. Use one video per post for reliable behavior.
- **Link previews**: If your text content includes a URL, Facebook will automatically generate a link preview. This is Facebook's behavior and is not controlled by Publora.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 2,200 characters (Publora frontend limit; see note below) |
| Comment | 8,000 characters |

> **Note:** The 2,200-character limit is enforced by the Publora frontend/editor only. The API does not enforce this limit — content up to Facebook's native 63,206-character limit will be accepted.

## API Limits

<!-- limits tables below synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->

These limits apply when posting via the Facebook Graph API (and by extension, Publora).

### Character Limit

- **2,200 characters** maximum for post body via the Publora frontend/editor. The API does not enforce this limit; content up to Facebook's native 63,206-character limit will be accepted.
- Posts under 80 characters get 66% more engagement

### Image Limits (API)

| Property | Limit |
|----------|-------|
| Max size | 10 MB (PNG recommended max 1 MB to avoid pixelation) |
| Max count | Up to 10 images per post |
| Formats | JPEG, PNG, GIF, BMP, TIFF, WebP, AVIF, HEIF, HEIC |

### Video Limits (API)

| Property | API Limit | Native Limit |
|----------|-----------|--------------|
| Duration | 45 minutes max | 240 min (4 hours) |
| Max size | 2 GB through the presigned API upload flow | 4 GB |
| Formats | MP4, MOV | - |

**Reels via API:**
- Duration: 90 seconds max
- Pages only (not personal profiles)

### Important API Restrictions

- **Reels can only be posted to Pages** (not personal profiles)
- **Platform-side quotas:** Account-dependent and subject to change; not a Publora numeric contract.

### Common Error Messages

| Error Code | Description |
|------------|-------------|
| `Error 1363026` | Video exceeds 45 min duration |
| `Error 1363023` | File size exceeds Facebook's 2 GB API limit |
| `Error 1363128` | Reels duration outside 3-90 second range |

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
