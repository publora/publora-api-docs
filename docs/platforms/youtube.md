# YouTube API - Upload Video via REST API

Upload videos to YouTube programmatically using the Publora REST API. A simpler alternative to the official YouTube Data API, YouTube API client libraries, or Google APIs.

## YouTube API Overview

Publora provides a unified REST API for uploading and publishing videos to YouTube with configurable privacy settings (public, unlisted, private) and metadata (title, description). No need to manage Google OAuth flows, handle resumable uploads, or navigate YouTube API quotas.

### Why Use Publora Instead of YouTube Data API?

| Feature | Publora API | YouTube Data API |
|---------|-------------|------------------|
| Authentication | Single API key | Complex Google OAuth 2.0 |
| Video upload | Simple REST endpoint | Resumable upload protocol |
| Quota management | Handled automatically | Manual quota tracking |
| Multi-platform | Post to 10 platforms | YouTube only |
| Setup time | 5 minutes | Hours (Google Cloud setup) |
| Privacy controls | Full support | Full support |

### Keywords: YouTube API, YouTube upload video API, YouTube Data API, upload to YouTube programmatically, YouTube video upload API, YouTube posting API, YouTube REST API, YouTube developer API, YouTube automation API, publish video YouTube API, YouTube content API

## Platform ID Format

```
youtube-{channelId}
```

Where `{channelId}` is your YouTube channel ID assigned during account connection via Google OAuth.

## Requirements

- A YouTube channel connected via Google OAuth through the Publora dashboard
- API key from Publora
- Video content is **required** (YouTube is a video platform)

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text only | No | YouTube requires a video |
| Images | No | Not supported as standalone posts |
| Videos | Yes | MP4 format |

## Platform-Specific Settings

YouTube supports a `platformSettings` object to control video privacy and title:

```json
{
  "platformSettings": {
    "youtube": {
      "privacy": "public",
      "title": "My Video Title"
    }
  }
}
```

| Setting | Values | Default | Description |
|---------|--------|---------|-------------|
| `privacy` | `"private"`, `"public"`, `"unlisted"` | `"public"` | Video visibility on YouTube |
| `title` | string | First 70 characters of `content` | Title displayed on the video page |

### Privacy Settings

| Value | Description |
|-------|-------------|
| `private` | Only you can view the video |
| `public` | Anyone can search for and view the video |
| `unlisted` | Anyone with the link can view, but it will not appear in search results |

### Title Behavior

If you do not specify a `title` in the platform settings, Publora automatically uses the first 70 characters of your `content` field as the video title. The full `content` is used as the video description.

## Examples

### Upload a Public Video

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'How to Build a REST API in 10 Minutes - A complete tutorial covering Express.js setup, routing, middleware, error handling, and deployment to production. Perfect for beginners who want to get started with backend development.',
    platforms: ['youtube-UCxxxxxxxx'],
    platformSettings: {
      youtube: {
        privacy: 'public',
        title: 'How to Build a REST API in 10 Minutes'
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
        'content': 'How to Build a REST API in 10 Minutes - A complete tutorial covering Express.js setup, routing, middleware, error handling, and deployment to production. Perfect for beginners who want to get started with backend development.',
        'platforms': ['youtube-UCxxxxxxxx'],
        'platformSettings': {
            'youtube': {
                'privacy': 'public',
                'title': 'How to Build a REST API in 10 Minutes'
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
    "content": "How to Build a REST API in 10 Minutes - A complete tutorial covering Express.js setup, routing, middleware, error handling, and deployment to production. Perfect for beginners who want to get started with backend development.",
    "platforms": ["youtube-UCxxxxxxxx"],
    "platformSettings": {
      "youtube": {
        "privacy": "public",
        "title": "How to Build a REST API in 10 Minutes"
      }
    }
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'How to Build a REST API in 10 Minutes - A complete tutorial covering Express.js setup, routing, middleware, error handling, and deployment to production. Perfect for beginners who want to get started with backend development.',
  platforms: ['youtube-UCxxxxxxxx'],
  platformSettings: {
    youtube: {
      privacy: 'public',
      title: 'How to Build a REST API in 10 Minutes'
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

> **Note:** YouTube requires a video. First create the post, then upload the video using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

### Upload an Unlisted Video

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Internal demo recording for the team. This video covers the new dashboard features and upcoming roadmap items. Please do not share this link externally.',
    platforms: ['youtube-UCxxxxxxxx'],
    platformSettings: {
      youtube: {
        privacy: 'unlisted',
        title: 'Internal Demo - Q1 Dashboard Features'
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
        'content': 'Internal demo recording for the team. This video covers the new dashboard features and upcoming roadmap items. Please do not share this link externally.',
        'platforms': ['youtube-UCxxxxxxxx'],
        'platformSettings': {
            'youtube': {
                'privacy': 'unlisted',
                'title': 'Internal Demo - Q1 Dashboard Features'
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
    "content": "Internal demo recording for the team. This video covers the new dashboard features and upcoming roadmap items. Please do not share this link externally.",
    "platforms": ["youtube-UCxxxxxxxx"],
    "platformSettings": {
      "youtube": {
        "privacy": "unlisted",
        "title": "Internal Demo - Q1 Dashboard Features"
      }
    }
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Internal demo recording for the team. This video covers the new dashboard features and upcoming roadmap items. Please do not share this link externally.',
  platforms: ['youtube-UCxxxxxxxx'],
  platformSettings: {
    youtube: {
      privacy: 'unlisted',
      title: 'Internal Demo - Q1 Dashboard Features'
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

### Upload with Auto-Generated Title

If you omit the `title` setting, Publora uses the first 70 characters of `content` as the title.

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Weekly Vlog: What I learned shipping 3 features in 5 days. This week was intense but incredibly productive. We managed to ship the new analytics dashboard, the team collaboration feature, and a complete redesign of the onboarding flow.',
    platforms: ['youtube-UCxxxxxxxx']
  })
});

const data = await response.json();
// Title will be: "Weekly Vlog: What I learned shipping 3 features in 5 days. This week"
// Description will be the full content
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
        'content': 'Weekly Vlog: What I learned shipping 3 features in 5 days. This week was intense but incredibly productive. We managed to ship the new analytics dashboard, the team collaboration feature, and a complete redesign of the onboarding flow.',
        'platforms': ['youtube-UCxxxxxxxx']
    }
)

data = response.json()
# Title will be: "Weekly Vlog: What I learned shipping 3 features in 5 days. This week"
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Weekly Vlog: What I learned shipping 3 features in 5 days. This week was intense but incredibly productive. We managed to ship the new analytics dashboard, the team collaboration feature, and a complete redesign of the onboarding flow.",
    "platforms": ["youtube-UCxxxxxxxx"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Weekly Vlog: What I learned shipping 3 features in 5 days. This week was intense but incredibly productive. We managed to ship the new analytics dashboard, the team collaboration feature, and a complete redesign of the onboarding flow.',
  platforms: ['youtube-UCxxxxxxxx']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

// Title auto-generated from first 70 chars of content
console.log(response.data);
```

## Platform Quirks

- **Video only**: YouTube does not support text-only or image-only posts through the API. A video file must always be included.
- **MP4 required**: Only MP4 video format is supported. Convert your videos to MP4 before uploading.
- **Auto-generated title**: When no `title` is specified in platform settings, Publora takes the first 70 characters of the `content` field. This may result in truncated titles, so it is recommended to explicitly set the title.
- **Content becomes description**: The full `content` text is used as the YouTube video description, while the `title` (or auto-generated title) is used as the video title.
- **Processing time**: After upload, YouTube processes the video before it becomes available. This can take from a few seconds to several minutes depending on video length and resolution.
- **Daily upload quota**: YouTube enforces daily upload quotas. If you hit the quota, Publora will return the error from the YouTube API.
- **Privacy changes**: You can upload as `private` or `unlisted` first, then manually change the privacy to `public` later through YouTube Studio.

## Character Limits

| Element | Limit |
|---------|-------|
| Video title | 100 characters |
| Video description | 5,000 characters |
| Auto-generated title | First 70 characters of content |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
