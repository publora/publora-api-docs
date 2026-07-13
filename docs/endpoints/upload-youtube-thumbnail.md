# Upload YouTube Thumbnail

Upload a custom thumbnail for a YouTube video post. Publora stores the image as tracked media and returns a `mediaId` + `url` that you then attach via `platformSettings.youtube.thumbnail` on [update-post](update-post.md). See [YouTube → Custom Thumbnail](../platforms/youtube.md#custom-thumbnail) for the full workflow and publish-time behavior.

A thumbnail **cannot** be set on `create-post` — the upload requires an existing `postGroupId`, so the post (draft or scheduled) must exist first.

## Endpoint

```
POST https://api.publora.com/api/v1/upload-youtube-thumbnail
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `Content-Type` | Yes | `multipart/form-data` |

## Request Body

Sent as `multipart/form-data`:

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `thumbnail` | file | Yes | The thumbnail image. **JPEG or PNG**, **2 MB max**, **min 640×360 px** (1280×720 / 16:9 recommended). |
| `postGroupId` | string | Yes | The post group to attach the thumbnail to. Must be an editable post (`draft` or `scheduled`) that you own and is not currently publishing. |

## Response

```json
{
  "success": true,
  "thumbnail": {
    "mediaId": "665f8a1b2c3d4e5f6a7b8c9d",
    "url": "https://media.publora.com/images/665f...-thumb.jpg"
  }
}
```

| Field | Description |
|-------|-------------|
| `success` | `true` when the thumbnail was uploaded and stored. |
| `thumbnail.mediaId` | ID of the created thumbnail media record. Pass as `platformSettings.youtube.thumbnail.mediaId`. |
| `thumbnail.url` | Public URL of the stored thumbnail. Pass as `platformSettings.youtube.thumbnail.url`. |

> **Two-step flow:** uploading does **not** attach the thumbnail by itself. After a successful upload, call [`PUT /update-post/{postGroupId}`](update-post.md) with `platformSettings.youtube.thumbnail` set to the returned `mediaId` and `url`. Only Publora-tracked media from this endpoint is accepted — arbitrary URLs are rejected. It is applied best-effort after the video processes; if YouTube rejects it (unverified channel, invalid image) the video still publishes without it.

## Usage

### cURL

```bash
# 1. Create the draft post (returns postGroupId)
POST_ID=$(curl -s -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"content":"My video description","platforms":["youtube-UCxxxxxxxx"]}' \
  | jq -r '.postGroupId')

# 2. Upload the thumbnail against that postGroupId
THUMB=$(curl -s -X POST https://api.publora.com/api/v1/upload-youtube-thumbnail \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -F "thumbnail=@./thumb.jpg" \
  -F "postGroupId=$POST_ID")

MEDIA_ID=$(echo "$THUMB" | jq -r '.thumbnail.mediaId')
THUMB_URL=$(echo "$THUMB" | jq -r '.thumbnail.url')

# 3. Attach it via update-post (then schedule as usual)
curl -X PUT https://api.publora.com/api/v1/update-post/$POST_ID \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{\"platformSettings\":{\"youtube\":{\"thumbnail\":{\"mediaId\":\"$MEDIA_ID\",\"url\":\"$THUMB_URL\"}}}}"
```

### JavaScript (fetch)

```javascript
// postGroupId from a prior create-post call
const form = new FormData();
form.append('thumbnail', thumbFile); // File/Blob — JPEG or PNG, <= 2 MB, >= 640x360
form.append('postGroupId', postGroupId);

const res = await fetch('https://api.publora.com/api/v1/upload-youtube-thumbnail', {
  method: 'POST',
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }, // do NOT set Content-Type — fetch sets the multipart boundary
  body: form,
});
const { thumbnail } = await res.json();

// Attach it, then schedule
await fetch(`https://api.publora.com/api/v1/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY, 'Content-Type': 'application/json' },
  body: JSON.stringify({ platformSettings: { youtube: { thumbnail } } }),
});
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| `400` | `postGroupId is required` | The `postGroupId` field was missing. |
| `400` | `Invalid postGroupId` | The `postGroupId` is not a valid ID. |
| `400` | `thumbnail file is required` | No `thumbnail` file part was sent. |
| `400` | `YouTube thumbnails must be JPEG or PNG images` | Unsupported file type, or the bytes aren't a valid JPEG/PNG. |
| `400` | `YouTube thumbnails must be 2 MB or smaller` | The file exceeds 2 MB. |
| `400` | `YouTube thumbnails must be at least 640x360 pixels` | The image is below the minimum dimensions. |
| `400` | `Cannot upload YouTube thumbnail for a non-editable post` | The post is published or failed. |
| `404` | `Post group not found` | No such post group for this account. |
| `409` | `Post group is currently being published; cannot upload YouTube thumbnail. Retry once publishing completes.` | The scheduler is mid-publish. |
| `500` | `Failed to upload YouTube thumbnail` | Storage or server error. |

## Notes

- **Clear a thumbnail** by sending `platformSettings.youtube.thumbnail` with an empty `url` via [update-post](update-post.md).
- **Verified channel required** for YouTube to accept a custom thumbnail; otherwise it is skipped at publish (the video still goes out).
- **Cleanup.** Replacing the thumbnail or deleting the post removes the stored file automatically.
