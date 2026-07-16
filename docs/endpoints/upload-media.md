# Upload Media

Upload images, videos, or PDF documents to attach to posts. Uses pre-signed S3 URLs for direct uploads.

## Endpoint

```
POST https://api.publora.com/api/v1/get-upload-url
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Set to `mcp` for MCP tool access |
| `Content-Type` | Yes | `application/json` |

> **MCP access:** If the `x-publora-client: mcp` header is sent, the server validates `entitlements.features.mcpAccess` and returns `403 "MCP access is not enabled for this account"` if the feature is not enabled.

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `fileName` | string | Yes | Name of the file (e.g., `photo.jpg`). The filename is sanitized before use: whitespace is trimmed, spaces are replaced with underscores, and special characters (except underscores, dots, and hyphens) are removed. For example, `my photo (1).jpg` becomes `my_photo_1.jpg`. |
| `contentType` | string | Yes | MIME type. Must begin with `image/` or `video/`, or equal `application/pdf`; otherwise the API returns `400 "Only video, image, and PDF files are allowed"`. |
| `postGroupId` | string | Yes | The post group to attach this media to. The attach helper verifies that it belongs to the effective API user. |
| `type` | string | No | `"image"`, `"video"`, or `"document"`. When omitted, the API infers it from `contentType` (including PDF → `document`). |

## Response

```json
{
  "success": true,
  "uploadUrl": "https://brandcraft-media.s3.amazonaws.com/images/...",
  "fileUrl": "https://brandcraft-media.s3.amazonaws.com/images/1710500000000-product-photo.jpg",
  "mediaId": "65f8a1b2c3d4e5f6a7b8c9d0"
}
```

| Field | Description |
|-------|-------------|
| `success` | `true` if the upload URL was generated successfully. |
| `uploadUrl` | Pre-signed S3 URL. Upload your file here via HTTP PUT. Expires in 1 hour. |
| `fileUrl` | The public URL where the file will be accessible after upload. |
| `mediaId` | The unique ID of the created media record. Use this to reference or delete the media. |

> **Internal fields:** Media records also store additional internal fields (`urlPure`, `mimeType`, `addedAt`, `filePath`, `uploadUrl`, `fileName` (S3 key), `metadata`) that are not returned by the API. Note that `mimeType` is a top-level field on the media record when created via the API `get-upload-url` endpoint, but the `process-video` endpoint stores `mimeType` inside the `metadata` subdocument instead. The `metadata` object contains: `size`, `width`, `height`, `aspectRatio`, `format` (defaults to `"unknown"` — never populated by any endpoint), `frameRate`, `codecName`, `bitRate`, and `duration`.

### Dashboard vs API Differences

The dashboard actively uses a session-authenticated endpoint (`/media/generate-upload-url`) with a closely related response:

| Aspect | API (`/api/v1/get-upload-url`) | Dashboard (`/media/generate-upload-url`) |
|--------|-------------------------------|------------------------------------------|
| **Required fields** | `fileName`, `contentType`, `postGroupId` | `fileName`, `contentType`, `postGroupId` |
| **Optional fields** | `type` (inferred when omitted) | `metadata` (object), `type` (inferred when omitted) |
| **Validation** | Accepts `image/*`, `video/*`, or `application/pdf` | Accepts `image/*`, `video/*`, or `application/pdf` |
| **Missing-field errors** | `"fileName, contentType, and postGroupId are required"` | `"fileName and contentType are required"`, or `"postGroupId is required"` |
| **Response format** | `{ success, uploadUrl, fileUrl, mediaId }` | `{ uploadUrl, key, mediaId }` |
| **Auth** | API key (`x-publora-key`) | Session cookie |

Either route may also return `postGroupDemoted` and `message` when attaching the new asset demotes a scheduled group to draft.

## Upload Flow

> **Important:** When uploading media, always create the post as a **draft first**, upload media, then schedule. This prevents the scheduler from processing the post before media upload completes.

### Recommended workflow (with media):

```
1. POST /create-post                → Create draft (no scheduledTime), get postGroupId
2. POST /get-upload-url        → Get pre-signed URL
3. PUT {uploadUrl}                   → Upload file to S3
4. PUT /update-post/:postGroupId     → Set status="scheduled" and scheduledTime
```

### Why this matters

If you create a post with `scheduledTime` set immediately, the scheduler may attempt to publish before your media upload completes — resulting in a failed post or missing media.

> **⚠ Attaching media demotes a scheduled post to draft.** If you call `get-upload-url` against a post that is already `scheduled`, the post is automatically demoted back to `draft` so the new media set gets re-validated. The response carries `postGroupDemoted: true` and a `message`. **You must re-schedule** it afterward with `PUT /update-post/:postGroupId` (`status: "scheduled"` + `scheduledTime`). Skipping the re-schedule is the most common cause of media that "uploaded fine" but never published — the post silently sits in `draft`. The same demote happens when you remove media with `DELETE /media/:mediaId`.

### Quick workflow (text-only posts):

```
1. POST /create-post              → Create with scheduledTime (no media needed)
```

## Supported Formats

The upload endpoint validates the broad MIME family; platform-specific formats and limits are enforced when the post is scheduled.

| Type | Formats | Max Size |
|------|---------|----------|
| Image | Any `image/*` at upload; platform validator decides publishability | No shared presigned-upload cap |
| Video | Any `video/*` at upload; platform validator decides publishability | No shared presigned-upload cap |
| Document | `application/pdf` | Platform/document rules apply |

**Note:** The 512 MB per file limit applies only to the server-side multipart upload endpoint (`process-video`), not to pre-signed URL uploads. Individual platforms may impose their own size limits at publish time.

**Note:** Upload acceptance does not promise that a platform publisher will accept the same file; consult the platform limits before scheduling.

## Examples

### Complete workflow: Post with image

#### JavaScript (fetch)

```javascript
const API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

// Step 1: Create draft post (no scheduledTime)
const postRes = await fetch(`${BASE_URL}/create-post`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'x-publora-key': API_KEY },
  body: JSON.stringify({
    content: 'Check out our new product! 🚀',
    platforms: ['twitter-123456789', 'linkedin-ABC123']
    // No scheduledTime = draft
  })
});
const { postGroupId } = await postRes.json();

// Step 2: Get upload URL
const urlRes = await fetch(`${BASE_URL}/get-upload-url`, {
  method: 'POST',
  headers: { 'Content-Type': 'application/json', 'x-publora-key': API_KEY },
  body: JSON.stringify({
    fileName: 'product-photo.jpg',
    contentType: 'image/jpeg',
    type: 'image',
    postGroupId
  })
});
const { uploadUrl, fileUrl, mediaId } = await urlRes.json();

// Step 3: Upload file to S3
const fileBuffer = await fs.promises.readFile('./product-photo.jpg');
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'Content-Type': 'image/jpeg' },
  body: fileBuffer
});
console.log(`Uploaded: ${fileUrl} (mediaId: ${mediaId})`);

// Step 4: Schedule the post
await fetch(`${BASE_URL}/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: { 'Content-Type': 'application/json', 'x-publora-key': API_KEY },
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: '2026-03-01T14:00:00.000Z'
  })
});
console.log('Post scheduled!');
```

#### Python (requests)

```python
import requests

API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'
headers = {'Content-Type': 'application/json', 'x-publora-key': API_KEY}

# Step 1: Create draft post
post_res = requests.post(f'{BASE_URL}/create-post', headers=headers, json={
    'content': 'Check out our new product! 🚀',
    'platforms': ['twitter-123456789', 'linkedin-ABC123']
})
post_group_id = post_res.json()['postGroupId']

# Step 2: Get upload URL
url_res = requests.post(f'{BASE_URL}/get-upload-url', headers=headers, json={
    'fileName': 'product-photo.jpg',
    'contentType': 'image/jpeg',
    'type': 'image',
    'postGroupId': post_group_id
})
data = url_res.json()

# Step 3: Upload file to S3
with open('product-photo.jpg', 'rb') as f:
    requests.put(data['uploadUrl'], headers={'Content-Type': 'image/jpeg'}, data=f)
print(f"Uploaded: {data['fileUrl']} (mediaId: {data['mediaId']})")

# Step 4: Schedule the post
requests.put(f'{BASE_URL}/update-post/{post_group_id}', headers=headers, json={
    'status': 'scheduled',
    'scheduledTime': '2026-03-01T14:00:00.000Z'
})
print('Post scheduled!')
```

#### cURL

```bash
API_KEY="YOUR_API_KEY"

# Step 1: Create draft post
POST_GROUP_ID=$(curl -s -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d '{
    "content": "Check out our new product! 🚀",
    "platforms": ["twitter-123456789", "linkedin-ABC123"]
  }' | jq -r '.postGroupId')

# Step 2: Get upload URL
UPLOAD_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/get-upload-url \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d "{
    \"fileName\": \"product-photo.jpg\",
    \"contentType\": \"image/jpeg\",
    \"type\": \"image\",
    \"postGroupId\": \"$POST_GROUP_ID\"
  }")
UPLOAD_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.uploadUrl')
FILE_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.fileUrl')
MEDIA_ID=$(echo "$UPLOAD_RESPONSE" | jq -r '.mediaId')

# Step 3: Upload file to S3
curl -X PUT "$UPLOAD_URL" \
  -H "Content-Type: image/jpeg" \
  --data-binary @product-photo.jpg

# Step 4: Schedule the post
curl -X PUT "https://api.publora.com/api/v1/update-post/$POST_GROUP_ID" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d '{
    "status": "scheduled",
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
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
const { uploadUrl, fileUrl, mediaId } = await urlResponse.json();

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
| 400 | `"fileName, contentType, and postGroupId are required"` | Missing `fileName`, `contentType`, or `postGroupId` |
| 400 | `"Invalid x-publora-user-id"` | The `x-publora-user-id` header value is not a valid ID |
| 401 | `"API key is required"` | `x-publora-key` header is missing entirely |
| 401 | `"Invalid API key"` | `x-publora-key` is present but invalid |
| 401 | `"Invalid API key owner"` | API key exists but the associated user account was not found |
| 403 | `"API access is not enabled for this account"` | The user's account does not have API access enabled |
| 403 | `"MCP access is not enabled for this account"` | The `x-publora-client: mcp` header was sent but `entitlements.features.mcpAccess` is not enabled |
| 403 | `"Workspace access is not enabled for this key"` | API key does not have workspace access enabled |
| 403 | `"User is not managed by key"` | Managed user does not belong to the API key owner's workspace |
| 500 | `"Failed to create upload URL"` | Internal server error during URL generation |

## Server-Side Video Upload

For video files, you can use the server-side upload endpoint which handles the upload and extracts video metadata (resolution, codec, frame rate, bitrate, duration).

> **Dashboard-only endpoint.** This endpoint uses session authentication (cookies) and is not available via API key auth. It is accessible only from the Publora dashboard.

### Endpoint

```
POST https://api.publora.com/media/process-video
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `Cookie` | Yes | Active dashboard session cookie |
| `Content-Type` | Yes | `multipart/form-data` |

### Request Body (multipart/form-data)

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `video` | file | Yes | The video file to upload |
| `postGroupId` | string | Yes | The post group to attach this video to |

This endpoint accepts `multipart/form-data` with a single video file attached (field name: `video`) and a `postGroupId`. The server uploads the file to S3 and extracts metadata automatically. Processing is asynchronous -- the endpoint returns a `sessionId` which can be used to track progress via SSE (Server-Sent Events).

> **Resolved issue:** The `process-video` endpoint now correctly passes `type: "video"` to the presigned URL generator. This was previously a bug where the missing `type` parameter resulted in an `undefined` S3 key, but it has been fixed.
>
> **Note:** The multer `fileFilter` rejection (for unsupported video formats) throws an error inside multer's callback. The resulting HTTP response format depends on Express's global error handler and may not produce a clean JSON 400 response.

### Response

```json
{
  "sessionId": "1710500000000"
}
```

The `sessionId` is returned for future use, but the SSE progress endpoint (`/processing-progress/:id`) is currently disabled (commented out in source). There is no working SSE endpoint to subscribe to yet.

**Limits:** 1 video file per request, 512 MB max. Accepted formats: MP4, MOV, AVI, MKV, and WebM.

### Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"No video file uploaded"` | No file is attached to the request |
| 400 | `"Unsupported video format. Allowed: MP4, MOV, AVI, MKV, WebM"` | The uploaded file format is not one of the accepted video formats. **Note:** This error is thrown inside multer's `fileFilter` callback, so it may not be returned as a clean JSON 400 response depending on Express's error handler. |

## Cleaning Up Abandoned Uploads

If a `POST /get-upload-url` call succeeds but the subsequent `PUT` to the presigned S3 URL is cancelled or fails, the MediaFile record is persisted to the post group. An abandoned upload would otherwise leave a broken reference that blocks re-scheduling with `MEDIA_NOT_READY`.

Two REST endpoints handle this by API key:

- **`DELETE /api/v1/media/:mediaId`** — delete a single media file. Detaches it from its post group and hard-deletes the row + S3 object. Returns `{ "success": true }`. If the parent post was `scheduled`, the response also includes `postGroupDemoted: true` and the group is demoted to `draft` (re-schedule with `update-post`). Errors: `400 "Invalid mediaId"` (malformed), `404 "Media file not found"`, `409` if the group is mid-publish, `400` if the group is already `published`/`failed`.
- **`DELETE /api/v1/delete-post/:postGroupId`** — removes the whole post group and all attached media (works in `draft` or `scheduled`).
- Or use the Publora dashboard.

## Finalizing an Upload (optional)

```
POST https://api.publora.com/api/v1/complete-media/:mediaFileId
```

After you `PUT` the bytes to the presigned URL, you *may* call `complete-media` to probe the file server-side immediately: it extracts authoritative metadata and advances the media from `uploading` → `ready` (or `failed`), giving you fast "is my media valid?" feedback. It is **optional** — the `update-post` scheduling gate probes any still-`uploading` media lazily when you schedule. Auth is the same `x-publora-key`.

Response:

```json
{
  "success": true,
  "mediaFile": { "_id": "...", "type": "image", "mimeType": "image/jpeg", "fileName": "images/...", "url": "https://...", "status": "ready", "metadata": { } }
}
```

> Note: `complete-media` only probes an already-attached file — it does **not** attach media and does **not** demote a scheduled post. Only `get-upload-url` (attach) and `DELETE /media` (detach) trigger the demote-to-draft behavior.

## File URLs

Uploaded file URLs use the S3 format and are returned directly as `fileUrl` in the response from the `get-upload-url` endpoint. For example:

```
https://your-bucket.s3.amazonaws.com/images/1710500000000-product-photo.jpg
```

> **Internal note:** The API internally stores two URL fields on each media record: `url` (the S3 domain URL, e.g., `https://your-bucket.s3.amazonaws.com/...`) and `urlPure` (the CDN URL via `media.publora.com`, e.g., `https://media.publora.com/...`). Only `url` is returned in API responses. The `urlPure` field is used internally for CDN-served media.

## Platform Media Limits

| Platform | Images | Videos | Notes |
|----------|--------|--------|-------|
| X / Twitter | Up to 4 | 1 per post | PNG preferred for images |
| LinkedIn | Multiple | 1 per post | WebP auto-converted to JPEG |
| Instagram | Carousel (10) | Reels or Stories | Publora connects Instagram accounts through Instagram Login for Business and requests `instagram_business_basic` plus `instagram_business_content_publish`; personal accounts are unsupported. Whether Meta accepts a particular Creator account is determined by Meta, not Publora's code. |
| Threads | Carousel | 1 per post | WebP auto-converted |
| TikTok | Up to 35 | 1 per post | Images: JPEG, PNG, WebP; video: MP4, MOV, WebM, min 23 FPS |
| YouTube | -- | 1 per post | MP4, streaming upload |
| Facebook | Multiple | 1 per post | Carousel support |
| Bluesky | Up to 4 | 1 per post | WebP auto-converted; alt text is not settable through REST |
| Mastodon | Up to 4 | 1 per post | Standard limits |
| Telegram | Multiple | 1 per post | 1024 char caption max (bot) |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
