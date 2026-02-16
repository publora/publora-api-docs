# Facebook

Publora integrates with Facebook Pages for publishing text posts, images (including carousels and albums), and video content. Multiple pages per account are supported.

## Platform ID Format

```
facebook-{pageId}
```

Where `{pageId}` is your Facebook Page ID assigned during account connection via Facebook OAuth. If you manage multiple pages, each page gets its own platform ID.

## Requirements

- A Facebook Page (not a personal profile) connected via OAuth through the Publora dashboard
- Page admin permissions granted during OAuth
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | No strict character limit (recommended under 63,206) |
| Images | Yes | Multiple supported (carousel/album), WebP auto-converted to JPEG |
| Videos | Yes | MP4 format |
| Multiple Pages | Yes | Each page has its own platform ID |

## Token Management

Facebook page tokens have a **59-day lifespan**. Publora automatically handles token refresh so you do not need to manually reconnect your account. However, if the refresh fails (e.g., due to permission changes on Facebook), you will need to reconnect the page through the Publora dashboard.

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
    platforms: ['facebook-112233445566'],
    mediaUrls: [
      'https://example.com/images/retreat-group.jpeg',
      'https://example.com/images/retreat-activity.jpeg',
      'https://example.com/images/retreat-dinner.jpeg',
      'https://example.com/images/retreat-sunset.jpeg'
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
        'content': 'Highlights from our company retreat last weekend! Great team, great memories.',
        'platforms': ['facebook-112233445566'],
        'mediaUrls': [
            'https://example.com/images/retreat-group.jpeg',
            'https://example.com/images/retreat-activity.jpeg',
            'https://example.com/images/retreat-dinner.jpeg',
            'https://example.com/images/retreat-sunset.jpeg'
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
    "content": "Highlights from our company retreat last weekend! Great team, great memories.",
    "platforms": ["facebook-112233445566"],
    "mediaUrls": [
      "https://example.com/images/retreat-group.jpeg",
      "https://example.com/images/retreat-activity.jpeg",
      "https://example.com/images/retreat-dinner.jpeg",
      "https://example.com/images/retreat-sunset.jpeg"
    ]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Highlights from our company retreat last weekend! Great team, great memories.',
  platforms: ['facebook-112233445566'],
  mediaUrls: [
    'https://example.com/images/retreat-group.jpeg',
    'https://example.com/images/retreat-activity.jpeg',
    'https://example.com/images/retreat-dinner.jpeg',
    'https://example.com/images/retreat-sunset.jpeg'
  ]
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

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
```

## Platform Quirks

- **Pages only, not profiles**: Publora posts to Facebook Pages, not personal profiles. The Facebook API does not allow posting to personal profiles via third-party apps.
- **Multiple pages**: If you manage multiple Facebook Pages, each page is connected separately and gets its own `facebook-{pageId}` platform ID. You can post to multiple pages in a single API call.
- **WebP auto-conversion**: WebP images are automatically converted to JPEG before uploading to Facebook. No action needed on your part.
- **59-day token lifespan**: Facebook page access tokens expire after 59 days. Publora auto-refreshes these tokens, but if the refresh fails, you will need to reconnect the page.
- **Album behavior**: When posting multiple images, Facebook may display them as a carousel or an album depending on the count and the viewer's device.
- **Video and images are separate**: A single post can contain either images or a video, but not both simultaneously.
- **Link previews**: If your text content includes a URL, Facebook will automatically generate a link preview. This is Facebook's behavior and is not controlled by Publora.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 63,206 characters (recommended) |
| Comment | 8,000 characters |
