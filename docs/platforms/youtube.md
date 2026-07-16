# YouTube API - Upload Video via REST API

Upload videos to YouTube programmatically using the Publora REST API. A simpler alternative to the official YouTube Data API, YouTube API client libraries, or Google APIs.

## YouTube API Overview

Publora provides a unified REST API for uploading and publishing videos to YouTube with configurable privacy (public, unlisted, private), metadata (title, description, tags, category, made-for-kids), YouTube playlists, and a tracked custom thumbnail. No need to manage Google OAuth flows, handle resumable uploads, or navigate YouTube API quotas.

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

> **Refresh token lifespan:** When the OAuth provider does not specify an expiry, Publora defaults the refresh token lifespan to approximately 182 days (~6 months), calculated as `365/2` days in the code. After that period, the user will need to re-authenticate via the Publora dashboard to reconnect their YouTube channel.

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text only | No | YouTube requires a video |
| Images | No | Not supported as standalone posts |
| Videos | Yes | MP4, MOV, AVI, WebM formats, single video only |

> **Important:** YouTube posts through Publora support only a single video per post. If you attempt to attach multiple videos, the API will reject the request. To publish multiple videos, create separate posts for each.

## Platform-Specific Settings

YouTube supports a `platformSettings` object for video metadata, a YouTube playlist, and a tracked custom thumbnail:

```json
{
  "platformSettings": {
    "youtube": {
      "privacy": "public",
      "title": "My video title",
      "tags": ["api", "tutorial"],
      "categoryId": "22",
      "madeForKids": false,
      "playlist": {
        "id": "PLxxxxxxxx",
        "platformId": "youtube-UCxxxxxxxx"
      },
      "thumbnail": {
        "mediaId": "665f...",
        "url": "https://media.publora.com/..."
      }
    }
  }
}
```

| Setting | Values | Default | Description |
|---------|--------|---------|-------------|
| `privacy` | `"private"`, `"public"`, `"unlisted"` | `"public"` | Video visibility on YouTube |
| `title` | string | `""` (empty string) | Title displayed on the video page. If empty, the publisher derives a title from the first 70 characters of the post content at publish time. |
| `tags` | string[] (or comma-separated string) | — | Video tags (`snippet.tags`). YouTube caps the **combined** length at 500 characters (tags containing whitespace are quote-wrapped, costing +2 chars each); the API trims the list to fit, so an oversized list can't fail the upload. Omitted entirely when empty. |
| `categoryId` | string | — | YouTube video category ID (e.g. `"22"` People & Blogs, `"28"` Science & Technology). When omitted, YouTube applies the channel's default. An invalid id will fail the upload — see [Category IDs](#category-ids). |
| `madeForKids` | boolean | `false` | Sets `status.selfDeclaredMadeForKids` — YouTube's required "made for kids" (COPPA) audience declaration. **Always sent** on every upload; defaults to `false` (not made for kids). |
| `playlist` | object `{ id, platformId }` | — | Add the published video to a YouTube playlist. `playlist.id` is the YouTube playlist id; `playlist.platformId` is the compound channel id (e.g. `youtube-UCxxxxxxxx`) and must match the single selected YouTube channel. Settable on `create-post`; changeable or clearable via `update-post`. See [Playlist](#playlist). |
| `thumbnail` | object `{ mediaId, url }` | — | Custom video thumbnail referencing a Publora-tracked media asset (uploaded via the dedicated thumbnail endpoint, which returns `mediaId` and `url`). Set via `update-post` (the upload requires a `postGroupId`). See [Custom Thumbnail](#custom-thumbnail). |

> **Note:** The full `content` is used as the video description. Pass `platformSettings` directly in the `create-post` request body — the API merges your values with the per-platform defaults.

### Tags

Pass `tags` as an array of strings (`["api", "tutorial"]`) or a comma-separated string (`"api, tutorial"`). The API trims blanks and enforces YouTube's 500-character combined cap by keeping tags until the budget is exhausted, so an over-long list is truncated rather than rejected.

### Made for kids

`madeForKids` maps to YouTube's required `selfDeclaredMadeForKids` audience declaration and is sent on **every** upload (defaulting to `false`). Set it to `true` if your content is directed at children, per YouTube/COPPA rules.

### Category IDs

`categoryId` accepts a YouTube category ID string. Common values: `"22"` (People & Blogs), `"28"` (Science & Technology), `"24"` (Entertainment), `"27"` (Education), `"10"` (Music). An invalid or region-disallowed id causes YouTube to reject the whole upload, so leave it unset to fall back to the channel default if unsure.

### Playlist

Add the published video to one of the channel's YouTube playlists.

```json
{
  "platformSettings": {
    "youtube": {
      "playlist": {
        "id": "PLxxxxxxxx",
        "platformId": "youtube-UCxxxxxxxx"
      }
    }
  }
}
```

- `playlist.id` — the YouTube playlist id.
- `playlist.platformId` — the compound YouTube platform id from `platforms` (e.g. `youtube-UCxxxxxxxx`). It must match the single selected YouTube channel.
- A playlist can only be set when **exactly one** YouTube channel is selected.
- Set the playlist on `create-post`, and change or clear it via `update-post`.
- The video is added to the playlist **after** the YouTube upload finishes processing.
- **Best-effort:** if adding the video to the playlist fails, the video stays published — only the playlist association is skipped.
- You **cannot** change the playlist while the post is already `processing` (returns `409`).

**Clear the playlist** by sending empty strings:

```json
{
  "platformSettings": {
    "youtube": {
      "playlist": {
        "id": "",
        "platformId": ""
      }
    }
  }
}
```

**Playlist errors:**

| Error | Cause |
|-------|-------|
| `platformSettings.youtube.playlistId is not supported; use platformSettings.youtube.playlist.` | Legacy `playlistId` field was sent — use the `playlist` object |
| `platformSettings.youtube.playlist must be an object.` | `playlist` is not an object |
| `platformSettings.youtube.playlist.id must be a string.` | `playlist.id` is not a string |
| `platformSettings.youtube.playlist.platformId must be a string.` | `playlist.platformId` is not a string |
| `YouTube playlist requires both playlist.id and playlist.platformId.` | One of the two fields is missing |
| `YouTube playlist can only be set when exactly one YouTube channel is selected.` | More than one (or no) YouTube channel is selected |
| `YouTube playlist platformId must match the selected YouTube channel.` | `playlist.platformId` does not match the chosen channel |
| `Post group is currently being published; cannot change YouTube playlist. Retry once publishing completes.` | The post is publishing/processing |

### Custom Thumbnail

A custom thumbnail is a **Publora-tracked media asset**, referenced by the `thumbnail` object:

```json
{
  "platformSettings": {
    "youtube": {
      "thumbnail": {
        "mediaId": "665f...",
        "url": "https://media.publora.com/..."
      }
    }
  }
}
```

**Flow:**

1. Create a draft post (`create-post`).
2. Upload the thumbnail image through the [dedicated YouTube thumbnail upload endpoint](../endpoints/upload-youtube-thumbnail.md) — it requires the `postGroupId`.
3. Use the returned `mediaId` and `url`.
4. Update the post with `platformSettings.youtube.thumbnail` (`update-post`).
5. Schedule the post.

> **Note:** A thumbnail **cannot** be passed on `create-post` — the thumbnail upload requires a `postGroupId`, so the post must exist first.

**Constraints:**

- **Format:** JPEG or PNG.
- **Max size:** 2 MB.
- **Min dimensions:** 640×360. **Recommended:** 1280×720 (16:9).
- **Must be Publora-uploaded / tracked media** — pass the `mediaId` and `url` returned by the thumbnail upload endpoint. Arbitrary URLs (including raw `media.publora.com` links) are not accepted; uploading via the generic media-upload flow is not sufficient.
- The custom thumbnail is applied **after** the video finishes processing, via the YouTube `thumbnails.set` API.
- **Best-effort:** if YouTube rejects the thumbnail (unverified channel, oversized/invalid image), **the video still publishes** — only the thumbnail is skipped.
- **Requires a verified YouTube channel.**
- You **cannot** change the thumbnail while the post is already `processing` (returns `409`).

**Clear the thumbnail** by sending an empty `url`:

```json
{
  "platformSettings": {
    "youtube": {
      "thumbnail": {
        "url": ""
      }
    }
  }
}
```

### Privacy Settings

| Value | Description |
|-------|-------------|
| `private` | Only you can view the video |
| `public` | Anyone can search for and view the video |
| `unlisted` | Anyone with the link can view, but it will not appear in search results |

### Title Behavior

If you do not specify a `title` in the platform settings, the API defaults the title to an empty string `""` at post creation. The publisher service may derive a title from the first 70 characters of the post content at publish time. The full `content` is used as the video description. Pass `platformSettings` directly in the `create-post` request body. The API merges user-provided settings with defaults per platform.

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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

### Upload with Auto-Generated Title

If you omit the `title` setting, the API defaults the title to an empty string `""` at post creation. The publisher service may derive a title from the first 70 characters of the post content at publish time. To set a custom title, include it in `platformSettings` when calling `create-post`.

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
// Note: title defaults to an empty string at post creation — the publisher service may derive one from content at publish time. Include platformSettings.youtube.title to set a custom one
// Description will be the full content
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Note: title defaults to an empty string at post creation — the publisher service may derive one from content at publish time. Include platformSettings.youtube.title to set a custom one
print(data)
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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

// Note: title defaults to an empty string at post creation — the publisher service may derive one from content at publish time. Include platformSettings.youtube.title to set a custom one
console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

## Platform Quirks

- **Video only**: YouTube does not support text-only or image-only posts through the API. A video file must always be included.
- **MP4, MOV, AVI, WebM supported**: Publora validates video format server-side using the platform limits configuration (`postValidationService.js` calls `isVideoFormatSupported(platform, mimeType)` for all platforms including YouTube). YouTube accepts MP4, MOV, AVI, and WebM formats. If you upload an unsupported format, Publora will reject it before it reaches the YouTube API.
- **Single video only**: Publora supports uploading one video per post. If you attempt to attach multiple videos, the API will reject the request with an error.
- **Default title is empty string**: When no `title` is specified in platform settings, the API defaults the title to an empty string `""` at post creation. The publisher service may derive a title from the first 70 characters of the post content at publish time. It is recommended to explicitly set the title via `platformSettings` in the `create-post` request for best results.
- **Content becomes description**: The full `content` text is used as the YouTube video description, while the `title` (or auto-generated title) is used as the video title.
- **Processing time**: After upload, YouTube processes the video before it becomes available. This can take from a few seconds to several minutes depending on video length and resolution.
- **Daily upload quota**: YouTube enforces daily upload quotas. If you hit the quota, Publora will return the error from the YouTube API.
- **Privacy changes**: You can upload as `private` or `unlisted` first, then manually change the privacy to `public` later through YouTube Studio.

## API Limits

<!-- limits tables below synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->

### Text Limits

| Element | Limit |
|---------|-------|
| Video title | 100 characters |
| Video description | 1,000 characters (frontend validation only; YouTube allows up to 5,000) |
| Default title | Empty string `""` at post creation (the publisher service may derive a title from the first 70 characters of post content at publish time) |
| Visible description | First 150 characters (shown before "Show more") |

### Media Limits

| Media Type | Max Size | Max Duration | Supported Formats |
|------------|----------|--------------|-------------------|
| Videos | 256 GB through the presigned API upload flow | 12 hours | MP4, MOV, AVI, WebM |
| Images | Not supported | - | - |

### Additional Notes

- YouTube is a video-only platform; images cannot be posted as standalone content
- The first 150 characters of the description are visible without clicking "Show more"
- Video processing time varies based on length and resolution
- Presigned API uploads support YouTube's 256 GB / 12-hour limit. The separate dashboard multipart video-processing route has a 512 MB multer cap; that cap does not apply to presigned API uploads.
- Publora enforces a 1,000-character limit on video descriptions via frontend validation only. The API itself does not enforce this limit, so API users can send up to YouTube's native 5,000-character limit.

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
