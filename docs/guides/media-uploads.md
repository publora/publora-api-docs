# Media Uploads

This guide covers how to upload images and videos to Publora using pre-signed S3 URLs, and how to attach media to your scheduled posts.

## How It Works

Publora uses a **pre-signed S3 URL workflow** for media uploads. This means your files are uploaded directly to cloud storage without passing through the Publora API server, ensuring fast and reliable transfers.

The workflow has three steps:

```
1. Request a pre-signed upload URL     POST /api/v1/media/get-upload-url
2. Upload the file directly to S3      PUT {presignedUrl}
3. Create a post referencing the media  POST /api/v1/post-groups
```

### Supported Formats

| Type | Formats |
|---|---|
| Images | JPEG, PNG, GIF, WebP |
| Videos | MP4, MOV |

### Limits

- **Maximum file size:** 512 MB per file
- **Per post:** Up to **4 images** OR **1 video** (not both)

### Automatic Processing

- **WebP images** are automatically converted to JPEG for platforms that do not support WebP natively.
- **Video metadata** is automatically extracted upon upload, including: resolution, codec, FPS, bitrate, duration, and aspect ratio. This metadata is used for platform-specific validation (e.g., TikTok FPS requirements).

## Examples

### Upload an Image and Create a Post

**JavaScript (fetch)**

```javascript
// Step 1: Get a pre-signed upload URL
const uploadUrlResponse = await fetch(
  'https://api.publora.com/api/v1/media/get-upload-url',
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      fileName: 'product-photo.jpg',
      contentType: 'image/jpeg'
    })
  }
);

const { presignedUrl, mediaKey } = await uploadUrlResponse.json();

// Step 2: Upload the file directly to S3
const fileBuffer = await fs.promises.readFile('./product-photo.jpg');

await fetch(presignedUrl, {
  method: 'PUT',
  headers: {
    'Content-Type': 'image/jpeg'
  },
  body: fileBuffer
});

// Step 3: Create a post with the uploaded media
const postResponse = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    text: 'Check out our latest product!',
    platformIds: ['twitter-123', 'linkedin-ABC', 'instagram-456'],
    mediaKeys: [mediaKey],
    scheduledTime: '2026-03-15T14:30:00.000Z'
  })
});

const post = await postResponse.json();
console.log('Post created with image:', post.id);
```

**Python (requests)**

```python
import requests

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

# Step 1: Get a pre-signed upload URL
upload_url_response = requests.post(
    f'{API_URL}/media/get-upload-url',
    headers=HEADERS,
    json={
        'fileName': 'product-photo.jpg',
        'contentType': 'image/jpeg'
    }
)

upload_data = upload_url_response.json()
presigned_url = upload_data['presignedUrl']
media_key = upload_data['mediaKey']

# Step 2: Upload the file directly to S3
with open('./product-photo.jpg', 'rb') as f:
    file_data = f.read()

requests.put(
    presigned_url,
    headers={'Content-Type': 'image/jpeg'},
    data=file_data
)

# Step 3: Create a post with the uploaded media
post_response = requests.post(
    f'{API_URL}/post-groups',
    headers=HEADERS,
    json={
        'text': 'Check out our latest product!',
        'platformIds': ['twitter-123', 'linkedin-ABC', 'instagram-456'],
        'mediaKeys': [media_key],
        'scheduledTime': '2026-03-15T14:30:00.000Z'
    }
)

post = post_response.json()
print(f"Post created with image: {post['id']}")
```

**cURL**

```bash
# Step 1: Get a pre-signed upload URL
UPLOAD_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/media/get-upload-url \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "fileName": "product-photo.jpg",
    "contentType": "image/jpeg"
  }')

PRESIGNED_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.presignedUrl')
MEDIA_KEY=$(echo "$UPLOAD_RESPONSE" | jq -r '.mediaKey')

# Step 2: Upload the file directly to S3
curl -X PUT "$PRESIGNED_URL" \
  -H "Content-Type: image/jpeg" \
  --data-binary @./product-photo.jpg

# Step 3: Create a post with the uploaded media
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d "{
    \"text\": \"Check out our latest product!\",
    \"platformIds\": [\"twitter-123\", \"linkedin-ABC\", \"instagram-456\"],
    \"mediaKeys\": [\"$MEDIA_KEY\"],
    \"scheduledTime\": \"2026-03-15T14:30:00.000Z\"
  }"
```

**Node.js (axios)**

```javascript
const axios = require('axios');
const fs = require('fs');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

// Step 1: Get a pre-signed upload URL
const { data: uploadData } = await api.post('/media/get-upload-url', {
  fileName: 'product-photo.jpg',
  contentType: 'image/jpeg'
});

const { presignedUrl, mediaKey } = uploadData;

// Step 2: Upload the file directly to S3
const fileBuffer = fs.readFileSync('./product-photo.jpg');

await axios.put(presignedUrl, fileBuffer, {
  headers: { 'Content-Type': 'image/jpeg' }
});

// Step 3: Create a post with the uploaded media
const { data: post } = await api.post('/post-groups', {
  text: 'Check out our latest product!',
  platformIds: ['twitter-123', 'linkedin-ABC', 'instagram-456'],
  mediaKeys: [mediaKey],
  scheduledTime: '2026-03-15T14:30:00.000Z'
});

console.log('Post created with image:', post.id);
```

---

### Upload a Video and Create a Post

**JavaScript (fetch)**

```javascript
const fs = require('fs');

// Step 1: Get a pre-signed upload URL for the video
const uploadUrlResponse = await fetch(
  'https://api.publora.com/api/v1/media/get-upload-url',
  {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      fileName: 'promo-video.mp4',
      contentType: 'video/mp4'
    })
  }
);

const { presignedUrl, mediaKey } = await uploadUrlResponse.json();

// Step 2: Upload the video to S3
const videoBuffer = await fs.promises.readFile('./promo-video.mp4');

await fetch(presignedUrl, {
  method: 'PUT',
  headers: { 'Content-Type': 'video/mp4' },
  body: videoBuffer
});

// Step 3: Create a post with the video
// Video metadata (resolution, codec, fps, bitrate, duration, aspect ratio)
// is extracted automatically by Publora after upload.
const postResponse = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    text: 'Watch our latest promo video!',
    platformIds: ['twitter-123', 'tiktok-789', 'youtube-012'],
    mediaKeys: [mediaKey],
    scheduledTime: '2026-03-15T16:00:00.000Z'
  })
});

const post = await postResponse.json();
console.log('Video post created:', post.id);
```

**Python (requests)**

```python
import requests

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

# Step 1: Get a pre-signed upload URL for the video
upload_url_response = requests.post(
    f'{API_URL}/media/get-upload-url',
    headers=HEADERS,
    json={
        'fileName': 'promo-video.mp4',
        'contentType': 'video/mp4'
    }
)

upload_data = upload_url_response.json()
presigned_url = upload_data['presignedUrl']
media_key = upload_data['mediaKey']

# Step 2: Upload the video to S3
with open('./promo-video.mp4', 'rb') as f:
    video_data = f.read()

requests.put(
    presigned_url,
    headers={'Content-Type': 'video/mp4'},
    data=video_data
)

# Step 3: Create a post with the video
post_response = requests.post(
    f'{API_URL}/post-groups',
    headers=HEADERS,
    json={
        'text': 'Watch our latest promo video!',
        'platformIds': ['twitter-123', 'tiktok-789', 'youtube-012'],
        'mediaKeys': [media_key],
        'scheduledTime': '2026-03-15T16:00:00.000Z'
    }
)

post = post_response.json()
print(f"Video post created: {post['id']}")
```

**cURL**

```bash
# Step 1: Get a pre-signed upload URL
UPLOAD_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/media/get-upload-url \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "fileName": "promo-video.mp4",
    "contentType": "video/mp4"
  }')

PRESIGNED_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.presignedUrl')
MEDIA_KEY=$(echo "$UPLOAD_RESPONSE" | jq -r '.mediaKey')

# Step 2: Upload the video to S3
curl -X PUT "$PRESIGNED_URL" \
  -H "Content-Type: video/mp4" \
  --data-binary @./promo-video.mp4

# Step 3: Create the video post
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d "{
    \"text\": \"Watch our latest promo video!\",
    \"platformIds\": [\"twitter-123\", \"tiktok-789\", \"youtube-012\"],
    \"mediaKeys\": [\"$MEDIA_KEY\"],
    \"scheduledTime\": \"2026-03-15T16:00:00.000Z\"
  }"
```

**Node.js (axios)**

```javascript
const axios = require('axios');
const fs = require('fs');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

// Step 1: Get a pre-signed upload URL
const { data: uploadData } = await api.post('/media/get-upload-url', {
  fileName: 'promo-video.mp4',
  contentType: 'video/mp4'
});

// Step 2: Upload the video to S3
const videoBuffer = fs.readFileSync('./promo-video.mp4');

await axios.put(uploadData.presignedUrl, videoBuffer, {
  headers: { 'Content-Type': 'video/mp4' },
  maxContentLength: 512 * 1024 * 1024, // 512 MB
  maxBodyLength: 512 * 1024 * 1024
});

// Step 3: Create the video post
const { data: post } = await api.post('/post-groups', {
  text: 'Watch our latest promo video!',
  platformIds: ['twitter-123', 'tiktok-789', 'youtube-012'],
  mediaKeys: [uploadData.mediaKey],
  scheduledTime: '2026-03-15T16:00:00.000Z'
});

console.log('Video post created:', post.id);
```

---

### Upload Multiple Images for a Carousel Post

You can attach up to 4 images to a single post. Each image requires its own upload URL.

**JavaScript (fetch)**

```javascript
const fs = require('fs');

const images = [
  { path: './slide1.jpg', name: 'slide1.jpg', type: 'image/jpeg' },
  { path: './slide2.png', name: 'slide2.png', type: 'image/png' },
  { path: './slide3.jpg', name: 'slide3.jpg', type: 'image/jpeg' },
  { path: './slide4.jpg', name: 'slide4.jpg', type: 'image/jpeg' }
];

const mediaKeys = [];

for (const image of images) {
  // Get upload URL for each image
  const uploadUrlResponse = await fetch(
    'https://api.publora.com/api/v1/media/get-upload-url',
    {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
      },
      body: JSON.stringify({
        fileName: image.name,
        contentType: image.type
      })
    }
  );

  const { presignedUrl, mediaKey } = await uploadUrlResponse.json();

  // Upload the file to S3
  const fileBuffer = await fs.promises.readFile(image.path);
  await fetch(presignedUrl, {
    method: 'PUT',
    headers: { 'Content-Type': image.type },
    body: fileBuffer
  });

  mediaKeys.push(mediaKey);
  console.log(`Uploaded ${image.name}`);
}

// Create a carousel post with all 4 images
const postResponse = await fetch('https://api.publora.com/api/v1/post-groups', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    text: 'Our product lineup for 2026 -- swipe to see all!',
    platformIds: ['twitter-123', 'linkedin-ABC', 'instagram-456'],
    mediaKeys: mediaKeys,
    scheduledTime: '2026-03-15T12:00:00.000Z'
  })
});

const post = await postResponse.json();
console.log('Carousel post created:', post.id);
```

**Python (requests)**

```python
import requests

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

images = [
    {'path': './slide1.jpg', 'name': 'slide1.jpg', 'type': 'image/jpeg'},
    {'path': './slide2.png', 'name': 'slide2.png', 'type': 'image/png'},
    {'path': './slide3.jpg', 'name': 'slide3.jpg', 'type': 'image/jpeg'},
    {'path': './slide4.jpg', 'name': 'slide4.jpg', 'type': 'image/jpeg'},
]

media_keys = []

for image in images:
    # Get upload URL
    upload_response = requests.post(
        f'{API_URL}/media/get-upload-url',
        headers=HEADERS,
        json={
            'fileName': image['name'],
            'contentType': image['type']
        }
    )

    upload_data = upload_response.json()

    # Upload to S3
    with open(image['path'], 'rb') as f:
        requests.put(
            upload_data['presignedUrl'],
            headers={'Content-Type': image['type']},
            data=f.read()
        )

    media_keys.append(upload_data['mediaKey'])
    print(f"Uploaded {image['name']}")

# Create the carousel post
post_response = requests.post(
    f'{API_URL}/post-groups',
    headers=HEADERS,
    json={
        'text': 'Our product lineup for 2026 -- swipe to see all!',
        'platformIds': ['twitter-123', 'linkedin-ABC', 'instagram-456'],
        'mediaKeys': media_keys,
        'scheduledTime': '2026-03-15T12:00:00.000Z'
    }
)

post = post_response.json()
print(f"Carousel post created: {post['id']}")
```

**cURL**

```bash
API_KEY="YOUR_API_KEY"
MEDIA_KEYS=()

# Upload 4 images
for FILE in slide1.jpg slide2.png slide3.jpg slide4.jpg; do
  CONTENT_TYPE="image/jpeg"
  if [[ "$FILE" == *.png ]]; then
    CONTENT_TYPE="image/png"
  fi

  UPLOAD_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/media/get-upload-url \
    -H "Content-Type: application/json" \
    -H "x-publora-key: $API_KEY" \
    -d "{
      \"fileName\": \"$FILE\",
      \"contentType\": \"$CONTENT_TYPE\"
    }")

  PRESIGNED_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.presignedUrl')
  MEDIA_KEY=$(echo "$UPLOAD_RESPONSE" | jq -r '.mediaKey')

  curl -s -X PUT "$PRESIGNED_URL" \
    -H "Content-Type: $CONTENT_TYPE" \
    --data-binary @"./$FILE"

  MEDIA_KEYS+=("\"$MEDIA_KEY\"")
  echo "Uploaded $FILE"
done

# Join media keys into JSON array
MEDIA_KEYS_JSON=$(IFS=,; echo "[${MEDIA_KEYS[*]}]")

# Create the carousel post
curl -X POST https://api.publora.com/api/v1/post-groups \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d "{
    \"text\": \"Our product lineup for 2026 -- swipe to see all!\",
    \"platformIds\": [\"twitter-123\", \"linkedin-ABC\", \"instagram-456\"],
    \"mediaKeys\": $MEDIA_KEYS_JSON,
    \"scheduledTime\": \"2026-03-15T12:00:00.000Z\"
  }"
```

**Node.js (axios)**

```javascript
const axios = require('axios');
const fs = require('fs');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

const images = [
  { path: './slide1.jpg', name: 'slide1.jpg', type: 'image/jpeg' },
  { path: './slide2.png', name: 'slide2.png', type: 'image/png' },
  { path: './slide3.jpg', name: 'slide3.jpg', type: 'image/jpeg' },
  { path: './slide4.jpg', name: 'slide4.jpg', type: 'image/jpeg' }
];

const mediaKeys = [];

for (const image of images) {
  // Get upload URL
  const { data: uploadData } = await api.post('/media/get-upload-url', {
    fileName: image.name,
    contentType: image.type
  });

  // Upload to S3
  const fileBuffer = fs.readFileSync(image.path);
  await axios.put(uploadData.presignedUrl, fileBuffer, {
    headers: { 'Content-Type': image.type }
  });

  mediaKeys.push(uploadData.mediaKey);
  console.log(`Uploaded ${image.name}`);
}

// Create the carousel post
const { data: post } = await api.post('/post-groups', {
  text: 'Our product lineup for 2026 -- swipe to see all!',
  platformIds: ['twitter-123', 'linkedin-ABC', 'instagram-456'],
  mediaKeys: mediaKeys,
  scheduledTime: '2026-03-15T12:00:00.000Z'
});

console.log('Carousel post created:', post.id);
```

## Best Practices

1. **Validate file size before uploading.** The maximum is 512 MB per file. Check the size client-side to avoid wasted bandwidth on uploads that will be rejected.

2. **Use the correct `contentType`.** The `contentType` you pass to `get-upload-url` must match the `Content-Type` header you send when uploading to S3. A mismatch will cause the upload to fail.

3. **Prefer JPEG over WebP.** While WebP is supported and will be auto-converted, starting with JPEG avoids the conversion step and ensures consistent quality across all platforms.

4. **Upload images in parallel when possible.** For carousel posts, you can request all 4 upload URLs at once and upload concurrently to speed things up.

5. **For large video files, consider streaming the upload.** Instead of reading the entire file into memory, use a stream-based approach for files approaching the 512 MB limit.

6. **Keep `mediaKey` references.** Store the `mediaKey` values returned from the upload step. You will need them when creating or updating posts.

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| S3 upload returns `403 Forbidden` | Pre-signed URL expired or `Content-Type` mismatch | Request a fresh upload URL and ensure the `Content-Type` header matches exactly |
| `400` when creating post with media | Too many images (>4) or mixing images and video | Use at most 4 images or exactly 1 video per post |
| WebP image looks different after posting | Auto-conversion to JPEG for incompatible platforms | Upload as JPEG directly if quality consistency is critical |
| Video post fails on TikTok | Video does not meet TikTok requirements (FPS, format, duration) | Check video metadata -- ensure proper FPS, supported codec, and acceptable duration |
| Upload is slow for large files | File being read entirely into memory | Use streaming upload for large video files |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
