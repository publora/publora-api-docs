# Upload Instagram Cover

Upload a custom cover image for an Instagram **Reel**. Publora hosts the image and returns a public URL you set as `platformSettings.instagram.coverUrl`. This is the file-upload alternative to hosting the JPEG yourself — see [Instagram → Platform-Specific Settings](../platforms/instagram.md#platform-specific-settings) for the `coverUrl` field.

The uploaded image is stored as a **JPEG** regardless of input format (Instagram accepts only JPEG for Reel covers); PNG and WebP are transcoded server-side, with transparency flattened onto a white background.

## Endpoint

```
POST https://api.publora.com/api/v1/upload-instagram-cover
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
| `cover` | file | Yes | The cover image. JPEG, PNG, or WebP, **8 MB max**. PNG/WebP are converted to JPEG. |
| `postGroupId` | string | Yes | The post group this cover belongs to. Must be an editable post (`draft` or `scheduled`) that you own. |

The target post group must be in `draft` or `scheduled` status and not currently publishing. A cover cannot be uploaded for a post that is already published, failed, or mid-publish.

## Response

```json
{
  "success": true,
  "cover": {
    "mediaId": "665f8a1b2c3d4e5f6a7b8c9d",
    "url": "https://media.publora.com/images/665f...-reel-cover.jpg"
  },
  "message": "Set platformSettings.instagram.coverUrl to cover.url via /update-post to use this image as the Reels cover."
}
```

| Field | Description |
|-------|-------------|
| `success` | `true` when the cover was uploaded and stored. |
| `cover.mediaId` | The unique ID of the created cover media record. |
| `cover.url` | The public JPEG URL. Set this as `platformSettings.instagram.coverUrl` via [update-post](update-post.md). |
| `message` | Reminder of the next step. |

> **Two-step flow:** uploading the cover does **not** attach it to the post by itself. After a successful upload, call [`PUT /update-post/{postGroupId}`](update-post.md) with `platformSettings.instagram.coverUrl` set to the returned `cover.url`. Deleting that media (or clearing `coverUrl`) reverts the Reel to the automatic/frame-based cover.

## Usage

### cURL

```bash
# 1. Upload the cover image
COVER_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/upload-instagram-cover \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -F "cover=@./reel-cover.png" \
  -F "postGroupId=507f1f77bcf86cd799439011")

COVER_URL=$(echo "$COVER_RESPONSE" | jq -r '.cover.url')

# 2. Attach it to the Reel
curl -X PUT https://api.publora.com/api/v1/update-post/507f1f77bcf86cd799439011 \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -d "{
    \"platformSettings\": {
      \"instagram\": { \"coverUrl\": \"$COVER_URL\" }
    }
  }"
```

### JavaScript (fetch)

```javascript
// 1. Upload the cover image
const form = new FormData();
form.append('cover', coverFile); // a File/Blob (JPEG, PNG, or WebP, <= 8 MB)
form.append('postGroupId', '507f1f77bcf86cd799439011');

const uploadRes = await fetch(
  'https://api.publora.com/api/v1/upload-instagram-cover',
  {
    method: 'POST',
    headers: { 'x-publora-key': process.env.PUBLORA_API_KEY },
    body: form, // do NOT set Content-Type — fetch sets the multipart boundary
  }
);
const { cover } = await uploadRes.json();

// 2. Attach it to the Reel
await fetch(
  'https://api.publora.com/api/v1/update-post/507f1f77bcf86cd799439011',
  {
    method: 'PUT',
    headers: {
      'x-publora-key': process.env.PUBLORA_API_KEY,
      'Content-Type': 'application/json',
    },
    body: JSON.stringify({
      platformSettings: { instagram: { coverUrl: cover.url } },
    }),
  }
);
```

### Python (requests)

```python
import requests

BASE_URL = "https://api.publora.com/api/v1"
headers = {"x-publora-key": PUBLORA_API_KEY}
post_group_id = "507f1f77bcf86cd799439011"

# 1. Upload the cover image
with open("reel-cover.png", "rb") as f:
    upload = requests.post(
        f"{BASE_URL}/upload-instagram-cover",
        headers=headers,
        files={"cover": ("reel-cover.png", f, "image/png")},
        data={"postGroupId": post_group_id},
    )
cover_url = upload.json()["cover"]["url"]

# 2. Attach it to the Reel
requests.put(
    f"{BASE_URL}/update-post/{post_group_id}",
    headers={**headers, "Content-Type": "application/json"},
    json={"platformSettings": {"instagram": {"coverUrl": cover_url}}},
)
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| `400` | `postGroupId is required` | The `postGroupId` field was missing. |
| `400` | `Invalid postGroupId` | The `postGroupId` is not a valid ID. |
| `400` | `cover file is required` | No `cover` file part was sent. |
| `400` | `Instagram covers must be JPEG, PNG, or WebP images` | Unsupported file type. |
| `400` | `Instagram covers must be 8 MB or smaller` | The uploaded file exceeds 8 MB. |
| `400` | `Instagram cover exceeds 8 MB after JPEG conversion; use a smaller image` | A dense PNG/WebP expanded past 8 MB when converted to JPEG. |
| `400` | `Cannot upload Instagram cover for a non-editable post` | The post is published or failed. |
| `404` | `Post group not found` | No such post group for this account. |
| `409` | `Post group is currently being published; cannot upload Instagram cover. Retry once publishing completes.` | The scheduler is mid-publish. |

## Notes

- **Reels only.** A cover applies to videos published as Reels (`videoType: "REELS"`, the default). It is ignored for Stories and image posts.
- **Precedence.** A cover set via `coverUrl` takes precedence over frame-based cover selection (`videoTimestamp`).
- **Cleanup.** Replacing or clearing the cover, or deleting the post, removes the stored cover file automatically.
