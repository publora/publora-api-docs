# Bluesky API / AT Protocol - Post to Bluesky via REST API

Post to Bluesky programmatically using the Publora REST API. A simpler alternative to the AT Protocol (atproto) SDK or direct Bluesky API integration.

## Bluesky API Overview

Publora provides a unified REST API for publishing text posts and media content to Bluesky. Supports rich text features like auto-detected hashtags and URLs. No need to manage AT Protocol complexity, handle Bluesky authentication flows, or implement the atproto SDK.

### Why Use Publora Instead of AT Protocol SDK / Bluesky API?

| Feature | Publora API | AT Protocol / Bluesky API |
|---------|-------------|--------------------------|
| Authentication | Single API key | App password + DID resolution |
| Rich text | Automatic facets | Manual facet creation |
| Multi-platform | Post to 10 platforms | Bluesky only |
| Setup time | 5 minutes | 30+ minutes |
| Media handling | Automatic | Manual blob upload |
| Rate limiting | Handled | Manual implementation |

### Keywords: Bluesky API, AT Protocol API, atproto API, Bluesky posting API, post to Bluesky programmatically, Bluesky REST API, Bluesky developer API, Bluesky automation API, Bluesky bot API, decentralized social API, Bluesky skeet API

## Platform ID Format

```
bluesky-{did}
```

Where `{did}` is your Bluesky Decentralized Identifier (DID), assigned during account connection.

## Requirements

- A Bluesky account connected via **identifier + app password** through the Publora dashboard. The `identifier` field is your Bluesky handle (e.g., `yourname.bsky.social`) — not labeled "username" in the connection form.
- You must use an **app password**, not your main account password (generate one in Bluesky Settings > App Passwords)
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 300 characters |
| Images | Yes | Up to 4 per post, all images converted to JPEG before upload |
| Videos | Yes | MP4 format |
| Alt text | Yes | Supported for images |
| Rich text | Yes | Hashtags and URLs auto-detected |

## Rich Text Facets

Bluesky uses a rich text system based on **facets** with byte offsets. Publora handles this complexity automatically:

- **Hashtags**: Any `#hashtag` in your content is automatically detected and converted to a clickable hashtag facet with correct byte offset calculation.
- **URLs**: Any URL in your content (e.g., `https://example.com`) is automatically detected and converted to a clickable link facet.
- **Byte offsets**: Bluesky requires precise byte offsets for facets, not character offsets. Publora calculates these correctly, even for content with multi-byte characters (e.g., emojis, non-Latin scripts).

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
    content: 'Just launched our new API documentation! Check it out at https://docs.example.com #devtools #api',
    platforms: ['bluesky-did:plc:abc123xyz']
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
        'content': 'Just launched our new API documentation! Check it out at https://docs.example.com #devtools #api',
        'platforms': ['bluesky-did:plc:abc123xyz']
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
    "content": "Just launched our new API documentation! Check it out at https://docs.example.com #devtools #api",
    "platforms": ["bluesky-did:plc:abc123xyz"]
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Just launched our new API documentation! Check it out at https://docs.example.com #devtools #api',
  platforms: ['bluesky-did:plc:abc123xyz']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with an Image and Alt Text

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic',
    platforms: ['bluesky-did:plc:abc123xyz'],
    altTexts: ['Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123..." }
```

> **Note:** To attach media to a Bluesky post, first create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

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
        'content': 'Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic',
        'platforms': ['bluesky-did:plc:abc123xyz'],
        'altTexts': ['Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time']
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
    "content": "Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic",
    "platforms": ["bluesky-did:plc:abc123xyz"],
    "altTexts": ["Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time"]
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic',
  platforms: ['bluesky-did:plc:abc123xyz'],
  altTexts: ['Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123..." }
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
    content: 'Before and after our office renovation. What a transformation!',
    platforms: ['bluesky-did:plc:abc123xyz'],
    altTexts: [
      'Office space before renovation showing old desks and dim lighting',
      'Office space after renovation with modern furniture and bright natural light'
    ]
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
        'content': 'Before and after our office renovation. What a transformation!',
        'platforms': ['bluesky-did:plc:abc123xyz'],
        'altTexts': [
            'Office space before renovation showing old desks and dim lighting',
            'Office space after renovation with modern furniture and bright natural light'
        ]
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
    "content": "Before and after our office renovation. What a transformation!",
    "platforms": ["bluesky-did:plc:abc123xyz"],
    "altTexts": [
      "Office space before renovation showing old desks and dim lighting",
      "Office space after renovation with modern furniture and bright natural light"
    ]
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Before and after our office renovation. What a transformation!',
  platforms: ['bluesky-did:plc:abc123xyz'],
  altTexts: [
    'Office space before renovation showing old desks and dim lighting',
    'Office space after renovation with modern furniture and bright natural light'
  ]
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

- **App password required**: You must use a Bluesky app password, not your main account password. Generate one at Settings > App Passwords in the Bluesky app.
- **All images converted to JPEG**: All uploaded images (including PNG, WebP, GIF, and other formats) are converted to JPEG via sharp before uploading. JPEG is the only format sent to Bluesky regardless of the input format.
- **Up to 4 images**: A maximum of 4 images can be attached to a single post.
- **Rich text auto-detection**: Publora automatically detects hashtags (`#tag`) and URLs in your content and creates the correct Bluesky facets with proper byte offsets. You do not need to do any special formatting.
- **Byte offset precision**: Bluesky facets use byte offsets, not character offsets. This means multi-byte characters (emojis, CJK characters, etc.) are handled correctly by Publora, but if you are debugging, be aware of this distinction.
- **Alt text mapping**: The `altTexts` array maps positionally to the media files uploaded via the [media upload workflow](../guides/media-uploads.md). The first alt text corresponds to the first uploaded image, and so on. If you provide fewer alt texts than images, the remaining images will have no alt text. **Note:** The `altTexts` parameter is not currently processed by the `create-post` API endpoint and will be silently ignored. Alt text support is available through the dashboard.
- **DID-based platform ID**: Unlike other platforms that use numeric IDs, Bluesky uses a DID (Decentralized Identifier) format like `did:plc:abc123xyz`.
- **`test-connection` may report missing credentials**: The platform connection validator checks for both `accessToken` and `username` fields, but Bluesky connections store a `password` (app password) instead of `accessToken`. As a result, calling `test-connection` for a Bluesky account will always report that the connection lacks credentials. This is a known limitation -- the connection will still work for posting.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 300 characters |
| Alt text | 2,000 characters per image (Bluesky limit; not enforced by Publora) |
| Images | Up to 4 per post |

## API Limits

<!-- limits tables below synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->

**Character Limit:** 300 characters (links count toward the limit in Publora — validation uses total `content.length`, not a link-aware count)

**Image Limits:**
- **Max size: exactly 2,000,000 bytes** (decimal, not 2 MiB)
- Max count: 4
- Input formats: JPEG, PNG, WebP (all converted to JPEG before upload to Bluesky). Additional formats supported by sharp (GIF, TIFF, BMP) are also accepted and converted to JPEG.

**Video Limits:**
- Max duration: 3 minutes
- Max size: 100 MiB (104,857,600 bytes)
- Formats: MP4 only
- **Daily limit: 25 videos per day** (sourced from `@publora/platform-limits`)
- Email verification required before video uploads

**Common Error Messages:**
- `429 Too Many Requests` - Rate limit exceeded
- Video job state `JOB_STATE_FAILED` - Processing failed

**Rate Limits:**
- 25 videos per day (sourced from `@publora/platform-limits`)

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
