# Bluesky

Publora integrates with Bluesky for publishing text posts and media content. Bluesky connections use username and app password authentication, and Publora supports rich text features like auto-detected hashtags and URLs.

## Platform ID Format

```
bluesky-{did}
```

Where `{did}` is your Bluesky Decentralized Identifier (DID), assigned during account connection.

## Requirements

- A Bluesky account connected via **username + app password** through the Publora dashboard
- You must use an **app password**, not your main account password (generate one in Bluesky Settings > App Passwords)
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 300 characters |
| Images | Yes | Up to 4 per post, JPEG supported, WebP auto-converted |
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
    mediaUrls: ['https://example.com/images/dashboard-preview.jpeg'],
    altTexts: ['Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time']
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
        'content': 'Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic',
        'platforms': ['bluesky-did:plc:abc123xyz'],
        'mediaUrls': ['https://example.com/images/dashboard-preview.jpeg'],
        'altTexts': ['Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time']
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
    "content": "Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic",
    "platforms": ["bluesky-did:plc:abc123xyz"],
    "mediaUrls": ["https://example.com/images/dashboard-preview.jpeg"],
    "altTexts": ["Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Our new dashboard is live! Here is a preview of the analytics view. #buildinpublic',
  platforms: ['bluesky-did:plc:abc123xyz'],
  mediaUrls: ['https://example.com/images/dashboard-preview.jpeg'],
  altTexts: ['Screenshot of the analytics dashboard showing charts for user growth, engagement rate, and revenue over time']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
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
    mediaUrls: [
      'https://example.com/images/office-before.jpeg',
      'https://example.com/images/office-after.jpeg'
    ],
    altTexts: [
      'Office space before renovation showing old desks and dim lighting',
      'Office space after renovation with modern furniture and bright natural light'
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
        'content': 'Before and after our office renovation. What a transformation!',
        'platforms': ['bluesky-did:plc:abc123xyz'],
        'mediaUrls': [
            'https://example.com/images/office-before.jpeg',
            'https://example.com/images/office-after.jpeg'
        ],
        'altTexts': [
            'Office space before renovation showing old desks and dim lighting',
            'Office space after renovation with modern furniture and bright natural light'
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
    "content": "Before and after our office renovation. What a transformation!",
    "platforms": ["bluesky-did:plc:abc123xyz"],
    "mediaUrls": [
      "https://example.com/images/office-before.jpeg",
      "https://example.com/images/office-after.jpeg"
    ],
    "altTexts": [
      "Office space before renovation showing old desks and dim lighting",
      "Office space after renovation with modern furniture and bright natural light"
    ]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Before and after our office renovation. What a transformation!',
  platforms: ['bluesky-did:plc:abc123xyz'],
  mediaUrls: [
    'https://example.com/images/office-before.jpeg',
    'https://example.com/images/office-after.jpeg'
  ],
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
```

## Platform Quirks

- **App password required**: You must use a Bluesky app password, not your main account password. Generate one at Settings > App Passwords in the Bluesky app.
- **WebP auto-conversion**: WebP images are automatically converted to JPEG before uploading. JPEG is the preferred format.
- **Up to 4 images**: A maximum of 4 images can be attached to a single post.
- **Rich text auto-detection**: Publora automatically detects hashtags (`#tag`) and URLs in your content and creates the correct Bluesky facets with proper byte offsets. You do not need to do any special formatting.
- **Byte offset precision**: Bluesky facets use byte offsets, not character offsets. This means multi-byte characters (emojis, CJK characters, etc.) are handled correctly by Publora, but if you are debugging, be aware of this distinction.
- **Alt text mapping**: The `altTexts` array maps positionally to the `mediaUrls` array. The first alt text corresponds to the first image, and so on. If you provide fewer alt texts than images, the remaining images will have no alt text.
- **DID-based platform ID**: Unlike other platforms that use numeric IDs, Bluesky uses a DID (Decentralized Identifier) format like `did:plc:abc123xyz`.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 300 characters |
| Alt text | 2,000 characters per image |
| Images | Up to 4 per post |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
