# Upload Media

Upload images and videos to attach to posts. Uses pre-signed S3 URLs for direct uploads.

## Endpoint

```
POST https://api.publora.com/api/v1/get-upload-url
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `Content-Type` | Yes | `application/json` |

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fileName` | string | Yes | Name of the file (e.g., `photo.jpg`) |
| `contentType` | string | Yes | MIME type (e.g., `image/jpeg`, `video/mp4`) |
| `type` | string | Yes | `"image"` or `"video"` |
| `postGroupId` | string | Yes | The post group to attach this media to |

## Response

```json
{
  "success": true,
  "uploadUrl": "https://brandcraft-media.s3.amazonaws.com/images/...",
  "fileUrl": "https://media.publora.com/images/...",
  "mediaId": "507f1f77bcf86cd799439099"
}
```

| Field | Description |
|-------|-------------|
| `uploadUrl` | Pre-signed S3 URL. Upload your file here via HTTP PUT. Expires in 1 hour. |
| `fileUrl` | Public URL of the file after upload. |
| `mediaId` | Media record ID for tracking. |

## Upload Flow

```
1. POST /api/v1/create-post       → Create post, get postGroupId
2. POST /api/v1/get-upload-url    → Get pre-signed URL (requires postGroupId)
3. PUT {uploadUrl}                 → Upload file to S3 (auto-attached via postGroupId)
```

## Supported Formats

| Type | Formats | Max Size |
|------|---------|----------|
| Image | JPEG, PNG, GIF, WebP | 512 MB per file |
| Video | MP4, MOV | 512 MB per file |

**Limits:** Up to 4 images OR 1 video per post.

**Note:** WebP images are automatically converted to JPEG for platforms that don't support WebP (LinkedIn, Telegram, Bluesky).

## Examples

### Upload an image

#### JavaScript (fetch)

```javascript
// Step 1: Get upload URL
const urlResponse = await fetch('https://api.publora.com/api/v1/get-upload-url', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    fileName: 'product-photo.jpg',
    contentType: 'image/jpeg',
    type: 'image',
    postGroupId: '507f1f77bcf86cd799439011'
  })
});
const { uploadUrl, fileUrl, mediaId } = await urlResponse.json();

// Step 2: Upload file to S3
const fileBuffer = await fs.promises.readFile('./product-photo.jpg');
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'Content-Type': 'image/jpeg' },
  body: fileBuffer
});

console.log(`Uploaded: ${fileUrl}`);
```

#### Python (requests)

```python
import requests

# Step 1: Get upload URL
url_response = requests.post(
    'https://api.publora.com/api/v1/get-upload-url',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'fileName': 'product-photo.jpg',
        'contentType': 'image/jpeg',
        'type': 'image',
        'postGroupId': '507f1f77bcf86cd799439011'
    }
)
data = url_response.json()

# Step 2: Upload file to S3
with open('product-photo.jpg', 'rb') as f:
    requests.put(
        data['uploadUrl'],
        headers={'Content-Type': 'image/jpeg'},
        data=f
    )

print(f"Uploaded: {data['fileUrl']}")
```

#### cURL

```bash
# Step 1: Get upload URL
RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/get-upload-url \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "fileName": "product-photo.jpg",
    "contentType": "image/jpeg",
    "type": "image",
    "postGroupId": "507f1f77bcf86cd799439011"
  }')

UPLOAD_URL=$(echo $RESPONSE | jq -r '.uploadUrl')

# Step 2: Upload file to S3
curl -X PUT "$UPLOAD_URL" \
  -H "Content-Type: image/jpeg" \
  --data-binary @product-photo.jpg
```

### Upload a video

#### JavaScript (fetch)

```javascript
// Step 1: Get upload URL for video
const urlResponse = await fetch('https://api.publora.com/api/v1/get-upload-url', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    fileName: 'promo-video.mp4',
    contentType: 'video/mp4',
    type: 'video',
    postGroupId: '507f1f77bcf86cd799439011'
  })
});
const { uploadUrl } = await urlResponse.json();

// Step 2: Upload video to S3
const videoBuffer = await fs.promises.readFile('./promo-video.mp4');
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'Content-Type': 'video/mp4' },
  body: videoBuffer
});
```

#### Python (requests)

```python
# Step 1: Get upload URL for video
url_response = requests.post(
    'https://api.publora.com/api/v1/get-upload-url',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'fileName': 'promo-video.mp4',
        'contentType': 'video/mp4',
        'type': 'video',
        'postGroupId': '507f1f77bcf86cd799439011'
    }
)
upload_url = url_response.json()['uploadUrl']

# Step 2: Upload video to S3
with open('promo-video.mp4', 'rb') as f:
    requests.put(upload_url, headers={'Content-Type': 'video/mp4'}, data=f)
```

### Upload multiple images for a carousel

```javascript
const images = ['photo1.jpg', 'photo2.jpg', 'photo3.jpg', 'photo4.jpg'];

for (const image of images) {
  // Get upload URL for each image
  const urlRes = await fetch('https://api.publora.com/api/v1/get-upload-url', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      fileName: image,
      contentType: 'image/jpeg',
      type: 'image',
      postGroupId: '507f1f77bcf86cd799439011'
    })
  });
  const { uploadUrl } = await urlRes.json();

  // Upload each image
  const fileBuffer = await fs.promises.readFile(`./${image}`);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'image/jpeg' },
    body: fileBuffer
  });
}
// All 4 images now attached to the post group
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"fileName is required"` | Missing fileName |
| 400 | `"contentType is required"` | Missing contentType |
| 400 | `"type is required"` | Missing type parameter |
| 400 | `"postGroupId is required"` | Missing postGroupId |
| 400 | `"Invalid content type"` | Not an image/* or video/* MIME type |
| 400 | `"Invalid type"` | type must be "image" or "video" |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 500 | `"Server error"` | Internal server error during URL generation |

## Platform Media Limits

| Platform | Images | Videos | Notes |
|----------|--------|--------|-------|
| X / Twitter | Up to 4 | 1 per post | PNG preferred for images |
| LinkedIn | Multiple | 1 per post | WebP auto-converted to JPEG |
| Instagram | Carousel (10) | Reels or Stories | Business account required |
| Threads | Carousel | 1 per post | WebP auto-converted |
| TikTok | -- | 1 per post | MP4 only, min 23 FPS |
| YouTube | -- | 1 per post | MP4, streaming upload |
| Facebook | Multiple | 1 per post | Carousel support |
| Bluesky | Up to 4 | 1 per post | WebP auto-converted, alt text |
| Mastodon | Up to 4 | 1 per post | Standard limits |
| Telegram | Multiple | 1 per post | 1024 char caption max (bot) |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
