# TikTok API - Upload Video via REST API

Upload videos to TikTok programmatically using the Publora REST API. A simpler alternative to the official TikTok Developer API, TikTok Business SDK, or TikAPI.

## TikTok API Overview

Publora provides a unified REST API for uploading and publishing videos to TikTok with granular privacy and interaction controls. No need to manage complex TikTok OAuth flows, handle video processing, or navigate TikTok's developer portal requirements.

### Why Use Publora Instead of TikTok Developer API / TikTok Business SDK?

| Feature | Publora API | TikTok Developer API |
|---------|-------------|----------------------|
| Authentication | Single API key | Complex OAuth 2.0 flow |
| Video upload | Simple REST endpoint | Multi-step upload process |
| App approval | Not required | TikTok app review required |
| Multi-platform | Post to 10 platforms | TikTok only |
| Setup time | 5 minutes | Weeks (app review) |
| Privacy controls | Full support | Full support |

### Keywords: TikTok API, TikTok upload video API, TikTok posting API, TikTok video API, upload to TikTok programmatically, TikTok bot API, TikTok automation API, TikTok REST API, TikTok developer API, TikTok content API, post video to TikTok API

## Platform ID Format

```
tiktok-{userId}
```

Where `{userId}` is your TikTok user ID assigned during account connection via OAuth.

## Requirements

- A TikTok account connected via OAuth through the Publora dashboard
- API key from Publora
- Video content is **required** (TikTok is a video-only platform)

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text only | No | TikTok requires a video |
| Images | No | Not supported as standalone posts |
| Videos | Yes | MP4 format, minimum 23 FPS |

## Platform-Specific Settings

TikTok has extensive publishing settings that you can control through the `platformSettings` object:

```json
{
  "platformSettings": {
    "tiktok": {
      "viewerSetting": "PUBLIC_TO_EVERYONE",
      "allowComments": true,
      "allowDuet": true,
      "allowStitch": true,
      "commercialContent": false,
      "brandOrganic": false,
      "brandedContent": false
    }
  }
}
```

### Viewer Settings

| Value | Description |
|-------|-------------|
| `PUBLIC_TO_EVERYONE` | Anyone can view the video |
| `MUTUAL_FOLLOW_FRIENDS` | Only mutual followers can view |
| `FOLLOWER_OF_CREATOR` | Only your followers can view |
| `SELF_ONLY` | Only you can view (draft-like behavior) |

### Interaction Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `allowComments` | boolean | `true` | Whether viewers can comment |
| `allowDuet` | boolean | `true` | Whether viewers can create Duets |
| `allowStitch` | boolean | `true` | Whether viewers can Stitch your video |

### Commercial Content Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `commercialContent` | boolean | `false` | Whether this is commercial content |
| `brandOrganic` | boolean | `false` | Organic brand promotion (your own brand) |
| `brandedContent` | boolean | `false` | Paid partnership or sponsored content |

> **Note**: If `brandOrganic` or `brandedContent` is `true`, then `commercialContent` must also be `true`.

## Examples

### Post a Video

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'How we built our startup in 60 seconds #startup #tech #coding',
    platforms: ['tiktok-99887766'],
    platformSettings: {
      tiktok: {
        viewerSetting: 'PUBLIC_TO_EVERYONE',
        allowComments: true,
        allowDuet: true,
        allowStitch: true,
        commercialContent: false,
        brandOrganic: false,
        brandedContent: false
      }
    }
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
        'content': 'How we built our startup in 60 seconds #startup #tech #coding',
        'platforms': ['tiktok-99887766'],
        'platformSettings': {
            'tiktok': {
                'viewerSetting': 'PUBLIC_TO_EVERYONE',
                'allowComments': True,
                'allowDuet': True,
                'allowStitch': True,
                'commercialContent': False,
                'brandOrganic': False,
                'brandedContent': False
            }
        }
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
    "content": "How we built our startup in 60 seconds #startup #tech #coding",
    "platforms": ["tiktok-99887766"],
    "platformSettings": {
      "tiktok": {
        "viewerSetting": "PUBLIC_TO_EVERYONE",
        "allowComments": true,
        "allowDuet": true,
        "allowStitch": true,
        "commercialContent": false,
        "brandOrganic": false,
        "brandedContent": false
      }
    }
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'How we built our startup in 60 seconds #startup #tech #coding',
  platforms: ['tiktok-99887766'],
  platformSettings: {
    tiktok: {
      viewerSetting: 'PUBLIC_TO_EVERYONE',
      allowComments: true,
      allowDuet: true,
      allowStitch: true,
      commercialContent: false,
      brandOrganic: false,
      brandedContent: false
    }
  }
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

> **Note:** TikTok requires a video. First create the post, then upload the video using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

### Post a Private Video with Restricted Interactions

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Preview of our upcoming feature for close friends only',
    platforms: ['tiktok-99887766'],
    platformSettings: {
      tiktok: {
        viewerSetting: 'MUTUAL_FOLLOW_FRIENDS',
        allowComments: true,
        allowDuet: false,
        allowStitch: false,
        commercialContent: false,
        brandOrganic: false,
        brandedContent: false
      }
    }
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
        'content': 'Preview of our upcoming feature for close friends only',
        'platforms': ['tiktok-99887766'],
        'platformSettings': {
            'tiktok': {
                'viewerSetting': 'MUTUAL_FOLLOW_FRIENDS',
                'allowComments': True,
                'allowDuet': False,
                'allowStitch': False,
                'commercialContent': False,
                'brandOrganic': False,
                'brandedContent': False
            }
        }
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
    "content": "Preview of our upcoming feature for close friends only",
    "platforms": ["tiktok-99887766"],
    "platformSettings": {
      "tiktok": {
        "viewerSetting": "MUTUAL_FOLLOW_FRIENDS",
        "allowComments": true,
        "allowDuet": false,
        "allowStitch": false,
        "commercialContent": false,
        "brandOrganic": false,
        "brandedContent": false
      }
    }
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Preview of our upcoming feature for close friends only',
  platforms: ['tiktok-99887766'],
  platformSettings: {
    tiktok: {
      viewerSetting: 'MUTUAL_FOLLOW_FRIENDS',
      allowComments: true,
      allowDuet: false,
      allowStitch: false,
      commercialContent: false,
      brandOrganic: false,
      brandedContent: false
    }
  }
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

## Platform Quirks

- **Video only**: TikTok does not support text-only or image-only posts through the API. A video must always be included.
- **Minimum 23 FPS**: Videos must have a frame rate of at least 23 frames per second. Videos below this threshold will be rejected by TikTok.
- **MP4 required**: Only MP4 video format is supported. Convert other formats before uploading.
- **Commercial content disclosure**: If your video promotes a brand or is part of a paid partnership, you must set the appropriate commercial content flags. Failing to do so may violate TikTok's community guidelines.
- **Brand content dependencies**: Setting `brandOrganic` or `brandedContent` to `true` requires `commercialContent` to also be `true`. Publora will return a validation error if this rule is violated.
- **Viewer setting restrictions**: Some viewer settings may limit the interaction options available. For example, `SELF_ONLY` posts cannot receive comments from others.
- **Processing time**: TikTok videos may take some time to process after upload. The post may not appear immediately on the profile.

## Character Limits

| Element | Limit |
|---------|-------|
| Video caption | 2,200 characters |
| Hashtags | Included in caption character count |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
