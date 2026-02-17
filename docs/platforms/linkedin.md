# LinkedIn

Publora integrates with LinkedIn for professional content publishing, including text posts, media attachments, analytics retrieval, and reaction management.

## Platform ID Format

```
linkedin-{profileId}
```

Where `{profileId}` is your LinkedIn profile identifier assigned during account connection via OAuth.

## Requirements

- A LinkedIn account connected via OAuth through the Publora dashboard
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 3,000 characters |
| Images | Yes | JPEG, PNG (WebP auto-converted to JPEG), multiple supported |
| Videos | Yes | MP4 format |
| Analytics | Yes | IMPRESSION, MEMBERS_REACHED, RESHARE, REACTION, COMMENT |
| Reactions | Yes | LIKE, PRAISE, EMPATHY, INTEREST, APPRECIATION, ENTERTAINMENT |

## Analytics

Publora can retrieve analytics for your LinkedIn posts. Available metrics:

| Metric | Description |
|--------|-------------|
| `IMPRESSION` | Number of times the post was displayed |
| `MEMBERS_REACHED` | Unique LinkedIn members who saw the post |
| `RESHARE` | Number of times the post was shared |
| `REACTION` | Total reactions on the post |
| `COMMENT` | Number of comments on the post |

## Reactions

LinkedIn supports a richer set of reactions than a simple "like":

| Reaction | Description |
|----------|-------------|
| `LIKE` | Standard like |
| `PRAISE` | Clapping hands / applause |
| `EMPATHY` | Heart / love |
| `INTEREST` | Thoughtful / insightful |
| `APPRECIATION` | Supportive |
| `ENTERTAINMENT` | Funny / laughing |

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
    content: 'Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.',
    platforms: ['linkedin-987654321']
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
        'content': 'Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.',
        'platforms': ['linkedin-987654321']
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
    "content": "Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.",
    "platforms": ["linkedin-987654321"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.',
  platforms: ['linkedin-987654321']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

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
    content: 'Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!',
    platforms: ['linkedin-987654321'],
    mediaUrls: [
      'https://example.com/images/hackathon-team.jpeg',
      'https://example.com/images/hackathon-demo.png'
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
        'content': 'Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!',
        'platforms': ['linkedin-987654321'],
        'mediaUrls': [
            'https://example.com/images/hackathon-team.jpeg',
            'https://example.com/images/hackathon-demo.png'
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
    "content": "Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!",
    "platforms": ["linkedin-987654321"],
    "mediaUrls": [
      "https://example.com/images/hackathon-team.jpeg",
      "https://example.com/images/hackathon-demo.png"
    ]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!',
  platforms: ['linkedin-987654321'],
  mediaUrls: [
    'https://example.com/images/hackathon-team.jpeg',
    'https://example.com/images/hackathon-demo.png'
  ]
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Check Post Analytics

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/post-analytics?postId=post-abc123&platform=linkedin-987654321', {
  method: 'GET',
  headers: {
    'x-publora-key': 'YOUR_API_KEY'
  }
});

const analytics = await response.json();
console.log(analytics);
// {
//   impressions: 4521,
//   membersReached: 3200,
//   reshares: 12,
//   reactions: 89,
//   comments: 15
// }
```

**Python (requests)**

```python
import requests

response = requests.get(
    'https://api.publora.com/api/v1/post-analytics',
    headers={
        'x-publora-key': 'YOUR_API_KEY'
    },
    params={
        'postId': 'post-abc123',
        'platform': 'linkedin-987654321'
    }
)

analytics = response.json()
print(analytics)
```

**cURL**

```bash
curl -G https://api.publora.com/api/v1/post-analytics \
  -H "x-publora-key: YOUR_API_KEY" \
  -d "postId=post-abc123" \
  -d "platform=linkedin-987654321"
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.get('https://api.publora.com/api/v1/post-analytics', {
  headers: {
    'x-publora-key': 'YOUR_API_KEY'
  },
  params: {
    postId: 'post-abc123',
    platform: 'linkedin-987654321'
  }
});

console.log(response.data);
```

## Platform Quirks

- **WebP auto-conversion**: If you provide a WebP image URL, Publora automatically converts it to JPEG before uploading to LinkedIn. No action needed on your part.
- **3,000-character limit**: LinkedIn enforces a strict 3,000-character limit for post text. Publora will return an error if your content exceeds this.
- **Multiple images**: LinkedIn supports posting multiple images at once. They will appear as a carousel or multi-image post depending on the count.
- **Rich text not supported via API**: LinkedIn's API does not support bold, italic, or other rich text formatting. Use plain text or Unicode characters for emphasis.
- **Analytics delay**: LinkedIn analytics may take up to 24 hours to fully populate. Querying immediately after posting will return partial data.
- **Hashtags**: LinkedIn hashtags are supported in the content body. They are treated as plain text but become clickable on the platform.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 3,000 characters |
| Comment | 1,250 characters |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
