# Instagram

Publora integrates with Instagram for publishing image posts, carousels, Reels, and Stories through the Instagram Graph API. A business or creator account is required.

## Platform ID Format

```
instagram-{accountId}
```

Where `{accountId}` is your Instagram Business or Creator account ID assigned during connection via Facebook OAuth.

## Requirements

- An **Instagram Business** or **Creator** account (personal accounts are not supported by the Instagram API)
- The Instagram account must be connected to a Facebook Page
- Connected via OAuth through the Publora dashboard
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text only | No | Instagram requires at least one image or video |
| Images | Yes | Up to 10 per carousel |
| Videos (Reels) | Yes | MP4 format, default video type |
| Videos (Stories) | Yes | MP4 format, requires `videoType: "STORIES"` setting |
| Carousels | Yes | 2-10 images or videos |

## Platform-Specific Settings

Instagram supports a `platformSettings` object to control video behavior:

```json
{
  "platformSettings": {
    "instagram": {
      "videoType": "REELS"
    }
  }
}
```

| Setting | Values | Default | Description |
|---------|--------|---------|-------------|
| `videoType` | `"REELS"`, `"STORIES"` | `"REELS"` | Determines how videos are published |

## Examples

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
    content: 'Sunset views from the office rooftop. #startup #views',
    platforms: ['instagram-11223344'],
    mediaUrls: ['https://example.com/images/sunset.jpeg']
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
        'content': 'Sunset views from the office rooftop. #startup #views',
        'platforms': ['instagram-11223344'],
        'mediaUrls': ['https://example.com/images/sunset.jpeg']
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
    "content": "Sunset views from the office rooftop. #startup #views",
    "platforms": ["instagram-11223344"],
    "mediaUrls": ["https://example.com/images/sunset.jpeg"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Sunset views from the office rooftop. #startup #views',
  platforms: ['instagram-11223344'],
  mediaUrls: ['https://example.com/images/sunset.jpeg']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post a Carousel

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: '10 tips for better code reviews. Swipe through to learn them all! #coding #devtips',
    platforms: ['instagram-11223344'],
    mediaUrls: [
      'https://example.com/images/tip-1.png',
      'https://example.com/images/tip-2.png',
      'https://example.com/images/tip-3.png',
      'https://example.com/images/tip-4.png',
      'https://example.com/images/tip-5.png'
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
        'content': '10 tips for better code reviews. Swipe through to learn them all! #coding #devtips',
        'platforms': ['instagram-11223344'],
        'mediaUrls': [
            'https://example.com/images/tip-1.png',
            'https://example.com/images/tip-2.png',
            'https://example.com/images/tip-3.png',
            'https://example.com/images/tip-4.png',
            'https://example.com/images/tip-5.png'
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
    "content": "10 tips for better code reviews. Swipe through to learn them all! #coding #devtips",
    "platforms": ["instagram-11223344"],
    "mediaUrls": [
      "https://example.com/images/tip-1.png",
      "https://example.com/images/tip-2.png",
      "https://example.com/images/tip-3.png",
      "https://example.com/images/tip-4.png",
      "https://example.com/images/tip-5.png"
    ]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: '10 tips for better code reviews. Swipe through to learn them all! #coding #devtips',
  platforms: ['instagram-11223344'],
  mediaUrls: [
    'https://example.com/images/tip-1.png',
    'https://example.com/images/tip-2.png',
    'https://example.com/images/tip-3.png',
    'https://example.com/images/tip-4.png',
    'https://example.com/images/tip-5.png'
  ]
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post a Reel

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Behind the scenes of building our product. #buildinpublic',
    platforms: ['instagram-11223344'],
    mediaUrls: ['https://example.com/videos/behind-scenes.mp4'],
    platformSettings: {
      instagram: {
        videoType: 'REELS'
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
        'content': 'Behind the scenes of building our product. #buildinpublic',
        'platforms': ['instagram-11223344'],
        'mediaUrls': ['https://example.com/videos/behind-scenes.mp4'],
        'platformSettings': {
            'instagram': {
                'videoType': 'REELS'
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
    "content": "Behind the scenes of building our product. #buildinpublic",
    "platforms": ["instagram-11223344"],
    "mediaUrls": ["https://example.com/videos/behind-scenes.mp4"],
    "platformSettings": {
      "instagram": {
        "videoType": "REELS"
      }
    }
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Behind the scenes of building our product. #buildinpublic',
  platforms: ['instagram-11223344'],
  mediaUrls: ['https://example.com/videos/behind-scenes.mp4'],
  platformSettings: {
    instagram: {
      videoType: 'REELS'
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

- **No text-only posts**: Instagram requires at least one image or video. Attempting to post text without media will return an error.
- **Business account required**: Personal Instagram accounts cannot be used with the API. You must convert to a Business or Creator account.
- **Facebook Page connection**: Your Instagram Business account must be linked to a Facebook Page. This is a requirement of the Instagram Graph API.
- **Carousel limits**: Carousels require between 2 and 10 media items. A single image is posted as a regular photo post, not a carousel.
- **Reel is the default**: When posting a video, Publora defaults to publishing it as a Reel. Set `videoType: "STORIES"` to post as a Story instead.
- **Stories disappear**: Stories are ephemeral and will disappear after 24 hours. This is standard Instagram behavior.
- **Image aspect ratios**: Instagram supports aspect ratios between 4:5 (portrait) and 1.91:1 (landscape). Images outside this range may be cropped.
- **Caption hashtags**: Hashtags are included in the caption text. There is no separate hashtags field.

## Character Limits

| Element | Limit |
|---------|-------|
| Caption | 2,200 characters |
| Hashtags | 30 per post |
| Carousel items | 2-10 media items |
