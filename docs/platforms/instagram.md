# Instagram API - Post to Instagram via REST API

Post to Instagram programmatically using the Publora REST API. A simpler alternative to the Instagram Graph API, Instagram Basic Display API, or Instagrapi.

## Instagram API Overview

Publora provides a unified REST API for publishing image posts, carousels, Reels, and Stories to Instagram through the Instagram Graph API. A business or creator account is required. No need to manage OAuth flows, handle the Instagram Content Publishing API complexity, or set up a Facebook Developer app.

### Why Use Publora Instead of Instagram Graph API / Instagrapi?

| Feature | Publora API | Instagram Graph API |
|---------|-------------|---------------------|
| Authentication | Single API key | Instagram Business Login (OAuth 2.0) |
| API access | Instant | Facebook app review |
| Reels & Stories | Supported | Supported |
| Multi-platform | Post to 11 platforms | Instagram only |
| Setup time | 5 minutes | Days to weeks |
| Carousel support | Yes | Yes |

### Keywords: Instagram API, Instagram posting API, Instagram Graph API, post to Instagram programmatically, Instagram REST API, Instagram developer API, Instagram automation API, Instagram Reels API, Instagram Stories API, Instagram content API, Instagrapi alternative, Instagram bot API

## Platform ID Format

```
instagram-{accountId}
```

Where `{accountId}` is your Instagram Business account ID assigned during connection via Instagram OAuth.

## Requirements

- An **Instagram Business** account is recommended (personal accounts are not supported)
- Creator accounts may also work, but Business is the recommended account type
- Connected via Instagram OAuth through the Publora dashboard
- API key from Publora

> **Important:** Business accounts are recommended but Creator accounts also work with `instagram_business_*` scopes. Personal accounts are not supported.

## API Limits

These are critical limits specific to the Instagram Graph API (different from native app limits):

### Character Limit

- **Caption:** 2,200 characters (first 125 characters visible before "more")

### Image Limits (API)

| Limit | Value |
|-------|-------|
| Max file size | 8 MB |
| Max count (carousel) | 10 images (native app allows 20) |
| **Formats** | **JPEG recommended** via API (PNG may work in practice) |

> **Warning:** You cannot mix images and videos in the same carousel via the API.

### Video Limits (API)

| Limit | Reels | Carousel Videos |
|-------|-------|-----------------|
| Max duration | **3 minutes (180 seconds)** via API (only 5-90s Reels eligible for Reels tab). Native app allows up to 15-20 minutes. | 60 seconds |
| Min duration | 3 seconds | 3 seconds |
| Max file size | 300 MB | 300 MB |
| Formats | MP4, MOV | MP4, MOV |

### Rate Limits

- **50 posts per 24 hours** (some accounts report a limit of 25)

### Important API Restrictions

The following features are **not available** via the Instagram Graph API:

- Personal accounts (Business recommended, Creator may also work)
- Shopping tags
- Branded content tags
- Filters and effects
- Music/audio additions
- Mixing images and videos in carousels

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text only | No | Instagram requires at least one image or video |
| Images | Yes | JPEG recommended via API (PNG may work in practice), max 8 MB, 10 per carousel |
| Videos (Reels) | Yes | MP4/MOV, max 3 minutes (180 seconds), max 300 MB |
| Videos (Stories) | Yes | MP4/MOV, requires `videoType: "STORIES"` setting |
| Carousels | Yes | 2-10 items (API limit; native app allows 20) |

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
| `videoTimestamp` | number (milliseconds) | — | Selects the video cover frame at the specified timestamp. **Important:** This must be set at the **top level** of the post group request body (not nested under `platformSettings.instagram`). The `thumbOffset` name under `platformSettings.instagram` has no effect. **Note:** This setting is only available via the dashboard `updatePostGroup` endpoint, not via the `create-post` API. |

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
    platforms: ['instagram-11223344']
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
        'content': 'Sunset views from the office rooftop. #startup #views',
        'platforms': ['instagram-11223344']
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
    "content": "Sunset views from the office rooftop. #startup #views",
    "platforms": ["instagram-11223344"]
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Sunset views from the office rooftop. #startup #views',
  platforms: ['instagram-11223344']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123..." }
```

> **Note:** Instagram requires media on every post. First create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`. For carousels, upload multiple files to the same `postGroupId`.

### Post a Carousel

Carousels require 2-10 images. The workflow is: create a draft post, upload images, then schedule.

**JavaScript (fetch)**

```javascript
const API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

// Step 1: Create a draft post (no scheduledTime)
const postResponse = await fetch(`${BASE_URL}/create-post`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    content: '10 tips for better code reviews. Swipe through! #coding #devtips',
    platforms: ['instagram-11223344']
    // No scheduledTime = draft
  })
});

const { postGroupId } = await postResponse.json();

// Step 2: Upload each image (2-10 images for Instagram carousels)
const images = ['tip1.jpg', 'tip2.jpg', 'tip3.jpg'];

for (const fileName of images) {
  // Get upload URL
  const uploadUrlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': API_KEY
    },
    body: JSON.stringify({
      fileName,
      contentType: 'image/jpeg',
      type: 'image',
      postGroupId
    })
  });

  const { uploadUrl } = await uploadUrlResponse.json();

  // Upload to S3
  const fileBuffer = await fs.promises.readFile(`./${fileName}`);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'image/jpeg' },
    body: fileBuffer
  });
}

// Step 3: Schedule the post
await fetch(`${BASE_URL}/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: '2026-03-15T14:00:00.000Z'
  })
});

console.log('Carousel scheduled!');
```

**Python (requests)**

```python
import requests

API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
}

# Step 1: Create a draft post (no scheduledTime)
post_response = requests.post(
    f'{BASE_URL}/create-post',
    headers=HEADERS,
    json={
        'content': '10 tips for better code reviews. Swipe through! #coding #devtips',
        'platforms': ['instagram-11223344']
        # No scheduledTime = draft
    }
)

post_group_id = post_response.json()['postGroupId']

# Step 2: Upload each image (2-10 images for Instagram carousels)
images = ['tip1.jpg', 'tip2.jpg', 'tip3.jpg']

for file_name in images:
    # Get upload URL
    upload_response = requests.post(
        f'{BASE_URL}/get-upload-url',
        headers=HEADERS,
        json={
            'fileName': file_name,
            'contentType': 'image/jpeg',
            'type': 'image',
            'postGroupId': post_group_id
        }
    )

    upload_url = upload_response.json()['uploadUrl']

    # Upload to S3
    with open(f'./{file_name}', 'rb') as f:
        requests.put(upload_url, headers={'Content-Type': 'image/jpeg'}, data=f.read())

# Step 3: Schedule the post
requests.put(
    f'{BASE_URL}/update-post/{post_group_id}',
    headers=HEADERS,
    json={
        'status': 'scheduled',
        'scheduledTime': '2026-03-15T14:00:00.000Z'
    }
)

print('Carousel scheduled!')
```

**cURL**

```bash
API_KEY="YOUR_API_KEY"

# Step 1: Create a draft post (no scheduledTime)
POST_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d '{
    "content": "10 tips for better code reviews. Swipe through! #coding #devtips",
    "platforms": ["instagram-11223344"]
  }')

POST_GROUP_ID=$(echo "$POST_RESPONSE" | jq -r '.postGroupId')

# Step 2: Upload each image (2-10 images for Instagram carousels)
for FILE in tip1.jpg tip2.jpg tip3.jpg; do
  UPLOAD_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/get-upload-url \
    -H "Content-Type: application/json" \
    -H "x-publora-key: $API_KEY" \
    -d "{
      \"fileName\": \"$FILE\",
      \"contentType\": \"image/jpeg\",
      \"type\": \"image\",
      \"postGroupId\": \"$POST_GROUP_ID\"
    }")

  UPLOAD_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.uploadUrl')

  curl -s -X PUT "$UPLOAD_URL" \
    -H "Content-Type: image/jpeg" \
    --data-binary @"./$FILE"
done

# Step 3: Schedule the post
curl -X PUT "https://api.publora.com/api/v1/update-post/$POST_GROUP_ID" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d '{
    "status": "scheduled",
    "scheduledTime": "2026-03-15T14:00:00.000Z"
  }'
```

> **Note:** For immediate publishing, set `scheduledTime` to the current time. For more details, see the [media upload workflow](../guides/media-uploads.md).

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
    platformSettings: {
      instagram: {
        videoType: 'REELS'
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
        'content': 'Behind the scenes of building our product. #buildinpublic',
        'platforms': ['instagram-11223344'],
        'platformSettings': {
            'instagram': {
                'videoType': 'REELS'
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
    "content": "Behind the scenes of building our product. #buildinpublic",
    "platforms": ["instagram-11223344"],
    "platformSettings": {
      "instagram": {
        "videoType": "REELS"
      }
    }
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Behind the scenes of building our product. #buildinpublic',
  platforms: ['instagram-11223344'],
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
// Response: { "success": true, "postGroupId": "abc123..." }
```

## Platform Quirks

- **No text-only posts**: Instagram requires at least one image or video. Attempting to post text without media will return an error.
- **Business account recommended**: Personal Instagram accounts cannot be used with the API. Business is recommended; Creator accounts may also work but are not fully tested.
- **Direct Instagram connection**: Publora connects directly to Instagram via Instagram Business Login. No Facebook Page is required.
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


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
