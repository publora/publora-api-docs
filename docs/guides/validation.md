# Post Validation

This guide covers how Publora validates posts before scheduling to prevent publish failures, including what gets validated, the validation response format, error codes, and best practices for avoiding validation errors.

## Overview

Posts are validated before scheduling to catch issues that would cause publishing failures on target platforms. Each social media platform has unique requirements for content length, media types, file sizes, and video durations. Publora validates your posts against these platform-specific limits and returns detailed error information so you can fix issues before scheduling.

Validation runs when you change a post's status to `scheduled` in the Publora dashboard. The REST API `create-post` endpoint does **not** trigger full validation -- it only performs basic field presence checks. See the "When Validation Occurs" section below for details.

## When Validation Occurs

Validation runs automatically on these API calls:

| Endpoint | Trigger |
|---|---|
| Dashboard update to `scheduled` status | When changing a post's status to `scheduled` in the dashboard |

> **Note:** The `POST /api/v1/create-post` API endpoint does **not** run post validation (`postValidationService.validatePostGroup`). It only performs basic field presence checks (content, platforms, scheduledTime format). Full validation only runs in the dashboard flow when updating a post to `scheduled` status. The `PUT /api/v1/update-post/:id` endpoint also does **not** invoke the validation service -- it only accepts `status` and `scheduledTime` fields (for rescheduling or changing draft/scheduled status).

If validation fails with blocking errors, the API returns a `400` status code with details about what needs to be fixed. Warnings are returned but do not block the operation.

## What Gets Validated

### Content Length

Each platform has a maximum character limit. Some platforms also have minimum requirements.

| Platform | Max Characters | Notes |
|---|---|---|
| **Twitter/X** | 280 (standard) / 25,000 (Premium) | Threading supported for long content |
| **Instagram** | 2,200 | First 125 chars visible before "more" |
| **Threads** | 500 (10,000 with text attachment) | Max 5 links per post |
| **TikTok** | 2,200 (API) | Native app allows 4,000 |
| **LinkedIn** | 3,000 | First 210 chars visible before "see more" |
| **YouTube** | 100 (title) / 5,000 (description) | First 150 chars of description visible |
| **Facebook** | 63,206 | Posts under 80 chars get higher engagement |
| **Mastodon** | 500 | Instance-configurable (some allow 5,000+) |
| **Bluesky** | 300 | Links count toward the limit in Publora |
| **Telegram** | 4,096 (users) / 1,024 (bot captions) | Bots limited to 1,024 for captions |
| **Pinterest** | 100 (title) / 800 (description) | Optimal description: 220-232 chars. *Pinterest is in the platform registry but publishing support is not yet fully available/tested.* |

### Media Requirements

Some platforms require media, while others support text-only posts.

| Platform | Requires Media | Requires Video | Supports Text-Only |
|---|---|---|---|
| **Twitter/X** | No | No | Yes |
| **Instagram** | Yes | No | No |
| **Threads** | No | No | Yes |
| **TikTok** | Yes | Yes | No |
| **LinkedIn** | No | No | Yes |
| **YouTube** | Yes | Yes | No |
| **Facebook** | No | No | Yes |
| **Mastodon** | No | No | Yes |
| **Bluesky** | No | No | Yes |
| **Telegram** | No | No | Yes |
| **Pinterest** | Yes | No | No |

### Media File Sizes

| Platform | Max Image Size | Max Video Size | Max Media Count |
|---|---|---|---|
| **Twitter/X** | 5 MB | 512 MB | 4 images OR 1 video |
| **Instagram** | 8 MB | 300 MB | 10 (API carousel) |
| **Threads** | 8 MB | 500 MB | 10 |
| **TikTok** | - | 4 GB | 1 video only |
| **LinkedIn** | 5 MB | 500 MB | 10 images OR 1 video |
| **YouTube** | - | 256 GB | 1 video only |
| **Facebook** | 10 MB | 2 GB (API) | 10 images OR 1 video |
| **Mastodon** | 16 MB | ~99 MB | 4 |
| **Bluesky** | 1 MB | 100 MB | 4 |
| **Telegram** | 10 MB | 50 MB (Bot API) | 10 |
| **Pinterest** | 20 MB | 1 GB | 5 (carousel) |

### Media Formats

| Platform | Supported Image Formats | Supported Video Formats |
|---|---|---|
| **Twitter/X** | JPEG, PNG, GIF, WebP | MP4, MOV |
| **Instagram** | JPEG, PNG, WebP (WebP auto-converted) | MP4, MOV |
| **Threads** | JPEG, PNG | MP4, MOV |
| **TikTok** | - | MP4, MOV, WebM |
| **LinkedIn** | JPEG, PNG, GIF | MP4 |
| **YouTube** | - | MP4, MOV, AVI, WebM |
| **Facebook** | JPEG, PNG, GIF, BMP, TIFF | MP4, MOV |
| **Mastodon** | JPEG, PNG, GIF, WebP | MP4, MOV, WebM |
| **Bluesky** | JPEG, PNG, WebP (max 2000x2000 px) | MP4 |
| **Telegram** | JPEG, PNG, GIF, WebP, BMP | MP4, MOV, AVI, MKV, WebM |
| **Pinterest** | JPEG, PNG, TIFF, BMP, GIF, WebP | MP4, MOV |

> **Note:** Instagram accepts JPEG, PNG, and WebP images (WebP is auto-converted to JPEG before publishing). Animated GIF, BMP, and TIFF will fail validation.

### Video Duration Limits

| Platform | Min Duration | Max Duration (API) | Notes |
|---|---|---|---|
| **Twitter/X** | - | 2 min (120s) | Native allows 2:20 |
| **Instagram Reels** | - | 3 min (180s) | Native allows 15-20 min |
| **Instagram Carousel** | - | 60s per video | - |
| **Threads** | - | 5 min | - |
| **TikTok** | 3 seconds | 10 min | Native allows 60 min |
| **LinkedIn** | - | 30 min | - |
| **YouTube** | - | 12 hours | - |
| **Facebook** | - | 45 min | Native allows 240 min |
| **Facebook Reels** | 3 seconds | 90 seconds | - |
| **Mastodon** | - | No limit* | *Limited by file size |
| **Bluesky** | - | 3 min | Daily limit: 25 videos OR 10 GB |
| **Telegram** | - | No limit | Limited by 50 MB file size |
| **Pinterest** | - | 15 min | - |

## Validation Response Format

> **Note:** The structured validation response below (with `validation.errors[]`, `validation.warnings[]`, and `validation.summary`) is returned by the **dashboard validation endpoint** only. The REST API `create-post` endpoint does not run the full validation service and will not return this format (it only performs basic field presence checks). The REST API `update-post` endpoint and MCP `update_post` tool also do not invoke the validator.

When validation errors occur, the API returns a `400` status code with a structured response:

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "instagram",
        "code": "MEDIA_REQUIRED",
        "message": "Instagram posts require media",
        "field": "media",
        "severity": "error"
      },
      {
        "platform": "tiktok",
        "code": "VIDEO_REQUIRED",
        "message": "TikTok posts require a video",
        "field": "video",
        "severity": "error"
      }
    ],
    "warnings": [
      {
        "platform": "twitter",
        "code": "CONTENT_TOO_LONG",
        "message": "Content exceeds Twitter/X's 280 character limit (currently 450)",
        "field": "content",
        "severity": "warning",
        "constraint": "maxLength",
        "currentValue": 450
      }
    ],
    "summary": {
      "affectedPlatforms": ["instagram", "tiktok", "twitter"],
      "errorCount": 2,
      "warningCount": 1
    }
  }
}
```

### Response Fields

| Field | Type | Description |
|---|---|---|
| `valid` | boolean | `true` if all platforms pass validation, `false` otherwise |
| `errors` | array | Blocking errors that must be fixed before scheduling |
| `warnings` | array | Non-blocking issues that may affect publishing |
| `summary.affectedPlatforms` | array | List of platform names with errors or warnings |
| `summary.errorCount` | number | Total number of errors |
| `summary.warningCount` | number | Total number of warnings |

### Error/Warning Object Fields

| Field | Type | Description |
|---|---|---|
| `platform` | string | Platform name (e.g., "twitter", "instagram") |
| `code` | string | Machine-readable error code |
| `message` | string | Human-readable error description |
| `field` | string | The field that caused the error (e.g., "content", "imageSize", "videoSize", "imageFormat", "videoDuration", "video", "settings") |
| `severity` | string | Either `"error"` (blocking) or `"warning"` (non-blocking) |
| `constraint` | string | (optional) String descriptor of the violated constraint (e.g., `"maxLength"`, `"maxCount"`, `"maxSize"`, `"maxDuration"`, `"requiredType"`) |
| `currentValue` | number/string | (optional) The current value that exceeded the constraint |
| `maxValue` | number/string | (optional) The maximum allowed value |
| `minValue` | number/string | (optional) The minimum required value |
| `suggestions` | string[] | (optional) Suggested fixes or alternative actions |

## Error Codes Reference

### Content Errors

| Code | Description | Resolution |
|---|---|---|
| `CONTENT_TOO_LONG` | Content exceeds the platform's character limit | Shorten the content or use threading where supported. **Note:** On threading-capable platforms (Twitter, Threads) with threading enabled, this is a **warning** (content will be auto-threaded). On other platforms or when threading is disabled, it is an **error**. |
| `CONTENT_TOO_SHORT` | Content is below the minimum required length | Add more content |

*Reserved -- not currently emitted by the validation service.*

| `CONTENT_REQUIRED` | Text content is required but missing | Add text content to the post |

*Reserved -- not currently emitted by the validation service.*
| `CONTENT_OR_MEDIA_REQUIRED` | Either text or media is needed | Add content or attach media |
| `THREAD_PART_TOO_LONG` | A single thread part exceeds the platform's per-part character limit | Break the thread part into smaller segments. (This code is defined in the `@publora/platform-limits` package. Threading validation may occur in a separate service.) |
| `INVALID_PLATFORM_CONTENT` | Content contains elements not supported by the platform | Remove unsupported content elements. (This error code is defined in the codebase but not currently emitted by the validation service.) |

### Media Errors

| Code | Description | Resolution |
|---|---|---|
| `MEDIA_REQUIRED` | Platform requires at least one media file | Attach an image or video |
| `MEDIA_SIZE_EXCEEDED` | File exceeds the platform's size limit | Compress or resize the file |
| `MEDIA_COUNT_EXCEEDED` | Too many media files attached | Reduce the number of files |
| `MEDIA_TYPE_NOT_SUPPORTED` | File format not supported by the platform | Convert to a supported format |
| `MEDIA_DIMENSIONS_INVALID` | Image dimensions outside allowed range | Resize the image |
| `IMAGES_NOT_SUPPORTED` | Platform is video-only (TikTok, YouTube) | Use a video instead |

*Reserved -- not currently emitted by the validation service.*

### Video Errors

| Code | Description | Resolution |
|---|---|---|
| `VIDEO_REQUIRED` | Platform requires a video (TikTok, YouTube) | Attach a video file |
| `VIDEO_DURATION_EXCEEDED` | Video is longer than the platform allows | Trim the video to meet the limit |
| `VIDEO_DURATION_TOO_SHORT` | Video is shorter than the minimum required | Use a longer video (min 3s for TikTok/FB Reels) |
| `VIDEO_NOT_SUPPORTED` | Platform does not support video | Remove the video or change target platforms |

*Reserved -- not currently emitted by the validation service.*

### Platform Errors

| Code | Description | Resolution |
|---|---|---|
| `PLATFORM_NOT_SUPPORTED` | Unknown or unsupported platform | Check the platform ID format |
| `PLATFORM_SETTING_REQUIRED` | A required platform-specific setting is missing | Provide the required setting |
| `PLATFORM_SETTING_INVALID` | A platform-specific setting has an invalid value | Correct the setting value |

### System Errors

| Code | Description | Resolution |
|---|---|---|
| `VALIDATION_SYSTEM_ERROR` | An internal error occurred during validation | Retry the request; if persistent, contact support |

## Examples

### Validation Error: Missing Media for Instagram

When posting to Instagram without media:

**Request:**

```json
{
  "content": "Check out our latest update!",
  "platforms": ["twitter-123", "instagram-456"]
}
```

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "instagram",
        "code": "MEDIA_REQUIRED",
        "message": "Instagram posts require media",
        "field": "media",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["instagram"],
      "errorCount": 1,
      "warningCount": 0
    }
  }
}
```

### Validation Error: Video Required for TikTok

When posting an image to TikTok (video-only platform):

**Request:**

```json
{
  "content": "New product announcement!",
  "platforms": ["tiktok-789"],
  "media": [{ "type": "image", "url": "https://..." }]
}
```

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "tiktok",
        "code": "VIDEO_REQUIRED",
        "message": "TikTok posts require a video",
        "field": "video",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["tiktok"],
      "errorCount": 1,
      "warningCount": 0
    }
  }
}
```

### Validation Error: Content Too Long

When content exceeds Twitter's 280 character limit:

> **Threading note:** This example shows `CONTENT_TOO_LONG` as an `"error"` with `severity: "error"`. However, on threading-capable platforms (Twitter/X, Threads) with threading enabled, this would be a **warning** (`severity: "warning"`) instead, because the content will be automatically split into a thread. The example below assumes threading is disabled.

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "twitter",
        "code": "CONTENT_TOO_LONG",
        "message": "Content exceeds Twitter/X's 280 character limit (currently 450)",
        "field": "content",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["twitter"],
      "errorCount": 1,
      "warningCount": 0
    }
  }
}
```

### Validation Error: Image Format Not Supported

When uploading an animated GIF to Instagram (only JPEG, PNG, and WebP are supported):

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "instagram",
        "code": "MEDIA_TYPE_NOT_SUPPORTED",
        "message": "Image format \"image/gif\" is not supported on Instagram",
        "field": "imageFormat",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["instagram"],
      "errorCount": 1,
      "warningCount": 0
    }
  }
}
```

### Validation Error: Video Duration Exceeded

When a video exceeds platform limits:

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "instagram",
        "code": "VIDEO_DURATION_EXCEEDED",
        "message": "Video duration (3m 0s) exceeds Instagram's 60s carousel limit",
        "field": "videoDuration",
        "severity": "error"
      },
      {
        "platform": "twitter",
        "code": "VIDEO_DURATION_EXCEEDED",
        "message": "Video duration (3m 0s) exceeds Twitter/X's 2m 0s limit",
        "field": "videoDuration",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["instagram", "twitter"],
      "errorCount": 2,
      "warningCount": 0
    }
  }
}
```

### Validation Error: File Size Exceeded

When an image exceeds Bluesky's strict 1 MB limit:

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "bluesky",
        "code": "MEDIA_SIZE_EXCEEDED",
        "message": "Image (2.5MB) exceeds Bluesky's 1.0MB limit",
        "field": "imageSize",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["bluesky"],
      "errorCount": 1,
      "warningCount": 0
    }
  }
}
```

### Multiple Platform Errors

When a single post has issues on multiple platforms:

**Response (400):**

```json
{
  "error": "Validation failed",
  "validation": {
    "valid": false,
    "errors": [
      {
        "platform": "instagram",
        "code": "MEDIA_TYPE_NOT_SUPPORTED",
        "message": "Image format \"image/gif\" is not supported on Instagram",
        "field": "imageFormat",
        "severity": "error"
      },
      {
        "platform": "bluesky",
        "code": "MEDIA_SIZE_EXCEEDED",
        "message": "Image (2.5MB) exceeds Bluesky's 1.0MB limit",
        "field": "imageSize",
        "severity": "error"
      },
      {
        "platform": "tiktok",
        "code": "VIDEO_REQUIRED",
        "message": "TikTok posts require a video",
        "field": "video",
        "severity": "error"
      }
    ],
    "warnings": [],
    "summary": {
      "affectedPlatforms": ["instagram", "bluesky", "tiktok"],
      "errorCount": 3,
      "warningCount": 0
    }
  }
}
```

## Best Practices

1. **Validate content length client-side.** Check character counts before sending the API request. Different platforms have different limits (Twitter: 280, LinkedIn: 3,000, Bluesky: 300).

2. **Always use JPEG for Instagram.** The Instagram API does not support PNG or GIF. Convert images to JPEG before uploading.

3. **Compress images for Bluesky.** Bluesky has a strict 1 MB limit. Use JPEG compression at 80-85% quality to stay under the limit.

4. **Check video durations before uploading.** Instagram Reels API maxes out at 3 minutes (180s), carousel videos at 60 seconds, Twitter at 2 minutes, TikTok at 10 minutes. Trim videos accordingly.

5. **Use separate media for different platform groups.** If posting to both image and video platforms, consider creating separate posts:
   - Image posts for Instagram, Twitter, LinkedIn
   - Video posts for TikTok, YouTube

6. **Handle validation errors gracefully in your UI.** Display platform-specific error messages so users know exactly what to fix.

7. **Consider platform requirements when designing content.** If targeting TikTok, always start with video. If targeting Instagram, prepare JPEG images.

8. **Keep Bluesky image dimensions under 2000x2000 pixels.** Larger images will fail validation.

9. **For Telegram bots, keep captions under 1,024 characters.** Bots cannot exceed this limit even though users can send 4,096 characters.

10. **Test video FPS for TikTok.** TikTok has minimum FPS requirements. Videos with very low frame rates will fail at publish time.

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| Instagram post fails with `MEDIA_TYPE_NOT_SUPPORTED` | Using PNG or GIF format | Convert to JPEG before uploading |
| TikTok post fails with `VIDEO_REQUIRED` | Posting images to a video-only platform | Use a video file instead |
| Bluesky post fails with `MEDIA_SIZE_EXCEEDED` | Image over 1 MB | Compress to 80-85% JPEG quality |
| Twitter post fails with `CONTENT_TOO_LONG` | Content over 280 characters | Shorten content or use threading |
| Instagram Reels fails with `VIDEO_DURATION_EXCEEDED` | Video over 3 minutes | Trim video to under 3 minutes (180s) |
| Multiple platforms fail with different errors | Content not optimized for all targets | Create platform-specific posts or fix each error |
| Validation passes but publish fails | Platform-side issues (rate limits, account restrictions) | Check platform-specific error messages in the post status |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
