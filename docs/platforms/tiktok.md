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
tiktok-{openId}
```

Where `{openId}` is the TikTok `open_id` assigned during account connection via OAuth. This is an app-scoped identifier (not the user's public TikTok user ID).

## Requirements

- A TikTok account connected via OAuth through the Publora dashboard
- API key from Publora
- Media is **required** — either a video or a photo carousel of up to 35 images (TikTok has no text-only posts)

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text only | No | TikTok requires media (a video or photo) |
| Images | Yes | Photo carousel, up to 35 images (JPEG, PNG, WebP) |
| Videos | Yes | MP4, MOV, WebM, AVI, MKV formats (Publora upload layer); TikTok's API may reject formats other than MP4/MOV/WebM, minimum 23 FPS |

## Platform-Specific Settings

TikTok has extensive publishing settings that you can control through the `platformSettings` object:

```json
{
  "platformSettings": {
    "tiktok": {
      "viewerSetting": "PUBLIC_TO_EVERYONE",
      "allowComments": true,
      "allowDuet": false,
      "allowStitch": false,
      "commercialContent": false,
      "brandOrganic": false,
      "brandedContent": false
    }
  }
}
```

### Viewer Settings

> **Default: `PUBLIC_TO_EVERYONE`.** When you create a post via the REST API without specifying `viewerSetting`, Publora applies `PUBLIC_TO_EVERYONE`. A value is effectively required — an empty string is rejected with *"TikTok requires selecting who can view your post"* — so always set `viewerSetting` explicitly in your `platformSettings.tiktok` object.
>
> The viewer settings an account can actually use are determined by TikTok from that account's own configuration. Public accounts support all four levels below. If the connected account is private, TikTok limits it to `SELF_ONLY` and rejects other values for that account — switch the account to public in the TikTok app to unlock the full range.

| Value | Description |
|-------|-------------|
| `PUBLIC_TO_EVERYONE` | Anyone can view the video |
| `MUTUAL_FOLLOW_FRIENDS` | Only mutual followers can view |
| `FOLLOWER_OF_CREATOR` | Only your followers can view |
| `SELF_ONLY` | Only you can view (draft-like behavior) |

### Interaction Settings

> Each flag is direct: `true` allows the interaction, `false` blocks it.
>
> **Note:** Duet and Stitch apply to video posts only. TikTok also honors the account's own permissions — if a creator has disabled Duet, Stitch, or comments at the account level, that takes precedence over the request.

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `allowComments` | boolean | `true` | Allow viewers to comment |
| `allowDuet` | boolean | `false` | Allow viewers to create Duets |
| `allowStitch` | boolean | `false` | Allow viewers to Stitch your video |

### Commercial Content Settings

| Setting | Type | Default | Description |
|---------|------|---------|-------------|
| `commercialContent` | boolean | `false` | Whether this is commercial content |
| `brandOrganic` | boolean | `false` | Organic brand promotion (your own brand) |
| `brandedContent` | boolean | `false` | Paid partnership or sponsored content |

> **Note**: If `commercialContent` is `true`, then at least one of `brandOrganic` or `brandedContent` must also be `true`. Publora will return a validation error if `commercialContent` is enabled without specifying which type of commercial content applies.

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
// Response: { "success": true, "postGroupId": "abc123..." }
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
# Response: { "success": true, "postGroupId": "abc123..." }
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
        "allowDuet": false,
        "allowStitch": false,
        "commercialContent": false,
        "brandOrganic": false,
        "brandedContent": false
      }
    }
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
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
// Response: { "success": true, "postGroupId": "abc123..." }
```

> **Note:** TikTok requires media — a video or a photo carousel. First create the post, then upload your media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

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
// Response: { "success": true, "postGroupId": "abc123..." }
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
# Response: { "success": true, "postGroupId": "abc123..." }
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
# Response: { "success": true, "postGroupId": "abc123..." }
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
// Response: { "success": true, "postGroupId": "abc123..." }
```

## Platform Quirks

- **Video only**: TikTok does not support text-only or image-only posts through the API. A video must always be included.
- **Minimum 23 FPS**: Videos must have a frame rate of at least 23 frames per second. Videos below this threshold will be rejected by TikTok.
- **Supported formats**: Publora's upload layer accepts MP4, MOV, WebM, AVI, and MKV video formats. However, TikTok's own API may reject formats other than MP4, MOV, and WebM, so those three are recommended for reliable publishing.
- **Commercial content disclosure**: If your video promotes a brand or is part of a paid partnership, you must set the appropriate commercial content flags. Failing to do so may violate TikTok's community guidelines.
- **Brand content dependencies**: Setting `commercialContent` to `true` requires at least one of `brandOrganic` or `brandedContent` to also be `true`. Publora will return a validation error if `commercialContent` is enabled without specifying which type applies.
- **Viewer setting restrictions**: Some viewer settings may limit the interaction options available. For example, `SELF_ONLY` posts cannot receive comments from others.
- **Processing time**: TikTok videos may take some time to process after upload. The post may not appear immediately on the profile.

## Creator Info

Before publishing, you can query the connected TikTok account's capabilities (max video duration, whether comments/duet/stitch are allowed by the creator's settings, etc.) using the creator info endpoint.

**Endpoint:** `POST /auth/tiktok/get-creator-info`

> **Important:** This is a **dashboard-only endpoint** that requires session authentication. It is **not** available as a public API endpoint and **cannot** be called with API key (`x-publora-key`) authentication. It is listed here for reference only.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | Your TikTok platform ID (e.g., `tiktok-abc123def`) |

**cURL (session auth required)**
```bash
curl -X POST https://api.publora.com/auth/tiktok/get-creator-info \
  -H "Content-Type: application/json" \
  --cookie "session=YOUR_SESSION_COOKIE" \
  -d '{
    "platformId": "tiktok-abc123def"
  }'
```

The response includes the creator's available privacy levels, whether comments/duet/stitch can be toggled, and the maximum video duration allowed for the account.

## API Limits

These limits apply specifically to the TikTok Content Posting API and may differ from native TikTok app limits.

### Character Limit

| Element | API Limit | Native App Limit |
|---------|-----------|------------------|
| Video caption | 2,200 characters | 4,000 characters |
| Hashtags | Included in caption character count | Included in caption character count |

### Video Limits

| Specification | API Limit | Native App Limit |
|---------------|-----------|------------------|
| Maximum duration | **10 minutes (600 seconds)** -- actual limit is fetched dynamically from TikTok's creator info API and may vary per account | 60 minutes |
| Minimum duration | 3 seconds | 3 seconds |
| Maximum file size | 4 GB | 4 GB |
| Supported formats | MP4, MOV, WebM (TikTok API); AVI, MKV also accepted by Publora upload layer | MP4, MOV, WebM |
| Minimum frame rate | 23 FPS | 23 FPS |

### Rate Limits

| Limit Type | Value |
|------------|-------|
| Posts per day | 15-20 posts |
| Videos per minute | Max 2 videos |

### Publishing Notes

- **Video or photo**: TikTok accepts a single video (MP4, MOV, WebM), or a photo carousel of up to 35 images (JPEG, PNG, WebP). Other image formats are transcoded automatically where possible.
- **Privacy is account-dependent**: The viewer settings available to an account are set by TikTok from that account's configuration (see [Viewer Settings](#viewer-settings)). Public accounts can post at any level, including `PUBLIC_TO_EVERYONE`.
- **Rate limits**: 15–20 posts per day and up to 2 videos per minute per account. Exceeding them returns `spam_risk_too_many_posts`.

### Common Error Messages

| Error Code | Description | Solution |
|------------|-------------|----------|
| `spam_risk_too_many_posts` | Daily post limit reached | Wait 24 hours before posting again |
| `duration_check_failed` | Video duration out of range | Ensure video is between 3 seconds and your account's max duration (default 10 minutes) |
| `file_format_check_failed` | Unsupported media format | Use MP4, MOV, or WebM for video, or JPEG, PNG, or WebP for photos. (The message may reference '.mp4' only, but these formats are all accepted.) |

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
