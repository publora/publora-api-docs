# Platform Limits Reference

This comprehensive guide documents the limits for all 10 social media platforms supported by Publora. Understanding these limits is essential for building reliable integrations with the Publora API.

**Last Updated:** March 2026

> **Fetch these live:** the same limits are available as JSON from [`GET /platform-limits`](../endpoints/platform-limits.md) — call it to drive validation instead of hard-coding the tables below.

## Overview

When building applications with the Publora API, it is critical to understand that **API limits often differ from native app limits**. Social media platforms impose different restrictions depending on whether content is posted through their official apps, web interfaces, or third-party APIs.

> **Note:** Publora defines 11 platform keys internally (including Pinterest), but only **10 platforms are actively supported**. Pinterest is listed for reference and is planned for a future release.

Publora validates your content against these API-specific limits before scheduling. This guide covers:

- Character limits for captions and descriptions
- Image formats, sizes, and counts
- Video duration, file size, and format requirements
- Platform-specific requirements (which platforms require media)
- Rate limits for posting frequency
- Common API error messages and validation codes

**Key Principle:** Always design your integration around the API limits documented here, not the limits you observe when posting manually through native apps.

---

## Critical API vs Native App Differences

The following table highlights the most significant differences between API and native app limits. These are the restrictions most likely to cause unexpected failures if you assume native app behavior.

| Platform | API Restriction | Native App Allows | Impact |
|----------|-----------------|-------------------|--------|
| **Twitter/X** | 2:20 (140s) video max | 2:20 (140s) video | Videos over 140s fail validation |
| **Instagram** | 15 min Reels, 10 carousel items, no animated GIF/BMP/TIFF | Platform UI may differ | Animated GIF/BMP/TIFF rejected; use JPEG, PNG, or WebP |
| **TikTok** | 10 min video, 2,200 char captions | 60 min video, 4,000 chars | Long captions truncated or rejected |
| **Facebook** | 45 min video, 2 GB files | 240 min video, 4 GB files | Large files will fail |
| **LinkedIn** | 500 MB video | 5 GB video | Videos over 500 MB will fail |
| **Telegram** | 50 MB files (Bot API) | 4 GB (user clients) | Large media uploads will fail |

---

## Character Limits

All platforms have character limits for text content. Some platforms have different limits for premium accounts or specific content types.

<!-- synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->
| Platform | Standard Limit | Premium/Special | Notes |
|----------|---------------|-----------------|-------|
| **Twitter/X** | 280 characters | 25,000 (Premium) | Threading supported for long content |
| **Instagram** | 2,200 characters | - | First 125 chars visible before "more" |
| **Threads** | 500 characters | 10,000 (text attachment) | Threading disabled; max 5 links per post |
| **TikTok** | 2,200 characters (API) | 4,000 (native app) | API enforces stricter limit |
| **LinkedIn** | 3,000 characters | - | First 210 chars visible before "see more" |
| **YouTube** | 100 (title) / 5,000 (description) | - | First 150 chars of description visible |
| **Facebook** | 63,206 characters | - | Posts under 80 chars get 66% more engagement |
| **Mastodon** | 500 characters | Instance-configurable | Some instances allow 5,000+ |
| **Bluesky** | 300 characters | - | Links count toward the limit in Publora |
| **Telegram** | 4,096 characters | 1,024 (bot caption) | Bots limited to 1,024 for media captions |
| **Pinterest** | 100 (title) / 800 (description) | 500 (ads) | **Not currently supported** -- planned for a future release |

### Character Limit Best Practices

1. **Design for the lowest common denominator** when cross-posting. If posting to Twitter and Threads, keep content under 280 characters.

2. **Use threading** for long-form content on Twitter/X. Meta Threads multi-part publishing is disabled, so keep its content within the single-post limit.

3. **Front-load important information** since most platforms truncate visible content with a "see more" link.

---

## Image Limits

Image requirements vary significantly across platforms. Pay particular attention to each platform's format and size rules; Instagram accepts JPEG, PNG, and WebP images, with WebP auto-converted before publishing.

<!-- synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->
| Platform | Max Size | Max Count | Supported Formats |
|----------|----------|-----------|-------------------|
| **Twitter/X** | 5 MB | 4 | JPEG, PNG, GIF, WebP |
| **Instagram** | 8 MB | 10 (API carousel) | **JPEG, PNG, WebP** (WebP auto-converted) |
| **Threads** | 8 MB | 20 | JPEG, PNG, WebP |
| **TikTok** | 20 MB | 35 | JPEG, PNG, WebP |
| **LinkedIn** | 36,152,320 pixels primary gate; 50 MB ceiling | 10 (multi-image) | JPEG, PNG, GIF, WebP |
| **YouTube** | - | 0 | Video only for uploads |
| **Facebook** | 10 MB | 10 | JPEG, PNG, GIF, BMP, TIFF, WebP, AVIF, HEIF, HEIC |
| **Mastodon** | 16 MB | 4 | JPEG, PNG, GIF, WebP, HEIF, HEIC, AVIF (instance-configurable) |
| **Bluesky** | exactly 2,000,000 bytes | 4 | JPEG, PNG, WebP |
| **Telegram** | 10 MB | 10 | JPEG, PNG, GIF, WebP, BMP |
| **Pinterest** | 20 MB | 1 | JPEG, PNG — connect-only; not publishable |

### Critical Image Notes

| Issue | Platform | Details |
|-------|----------|---------|
| **Image formats** | Instagram | JPEG, PNG, and WebP are accepted (WebP is auto-converted to JPEG before publishing). Animated GIF, BMP, and TIFF are not supported. |
| **Carousel Limit** | Instagram | API carousels are limited to 10 items (native app allows 20). |
| **No Mixed Media** | Instagram | Cannot mix images and videos in the same carousel via API. |
| **Strict Size Limit** | Bluesky | Hard 2,000,000-byte decimal limit. |
| **No Organic Carousels** | LinkedIn | Organic swipeable carousels are NOT supported via API (only sponsored content). |

### Server-Side Upload Limits

The dashboard-only `/media/process-video` multipart endpoint accepts one `video` file per request and caps it at 512 MB.

Presigned API uploads do not share that dashboard multipart cap; platform limits still apply.

### Image Processing in Publora

Publora automatically handles some image conversions:

- **WebP images** are automatically converted to JPEG for platforms that do not support WebP natively.
- **Oversized images** are validated and rejected with a clear error message.

---

## Video Limits

Video restrictions through APIs are often significantly more restrictive than native apps. This is one of the most common sources of failed posts.

<!-- synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->
| Platform | Max Duration | Max Size | Supported Formats |
|----------|--------------|----------|-------------------|
| **Twitter/X (API)** | 2 min 20s (140s) | 512 MB | MP4, MOV |
| **Instagram Reels (API)** | 15 minutes (900s) | 300 MB | MP4, MOV |
| **Instagram Carousel** | 60s per video | 300 MB | MP4, MOV |
| **Threads** | 5 min | 1 GB | MP4, MOV |
| **TikTok (API)** | 10 min | 4 GB | MP4, MOV, WebM |
| **LinkedIn (API)** | 30 min | 500 MB | MP4 |
| **YouTube** | 12 hours | 256 GB | MP4, MOV, AVI, WebM |
| **Facebook (API)** | 45 min | 2 GB | MP4, MOV |
| **Mastodon** | 24 hours | ~99 MB | MP4, MOV, WebM |
| **Bluesky** | 3 min | 100 MB | MP4 |
| **Telegram (Bot API)** | 24 hours | 50 MB | MP4, MOV, AVI, MKV, WebM |
| **Pinterest** | 15 min | 2 GB | MP4, MOV — connect-only; not publishable |

*Mastodon video has a 24-hour package cap and an instance-configurable file-size limit (~99 MB default).

### Platform-Specific Video Restrictions

#### Instagram API

| Restriction | API Limit | Native App Limit |
|-------------|-----------|------------------|
| Reels duration | 15 minutes (900s) | Platform UI may differ |
| Image format | JPEG, PNG, WebP | GIF, BMP, TIFF also supported |
| Carousel items | 10 | 20 |
| Account type | Publora connects Instagram accounts through Instagram Login for Business and requests `instagram_business_basic` plus `instagram_business_content_publish`; personal accounts are unsupported. Whether Meta accepts a particular Creator account is determined by Meta, not Publora's code. | Platform UI eligibility is controlled by Meta |
| Features | No shopping tags, branded content, filters, or music | Full feature set |

#### TikTok API

| Restriction | API Limit | Native App Limit |
|-------------|-----------|------------------|
| Video duration | 10 minutes | 60 minutes |
| Caption length | 2,200 characters | 4,000 characters |

#### Facebook API

| Restriction | API Limit | Native App Limit |
|-------------|-----------|------------------|
| Video duration | 45 minutes | 240 minutes |
| File size | 2 GB | 4 GB |
| Reels posting | Pages only (not personal profiles) | Both supported |

#### LinkedIn API

| Restriction | API Limit | Native App Limit |
|-------------|-----------|------------------|
| Video file size | 500 MB | 5 GB |
| Video duration | 30 minutes | 15 minutes (API allows more) |
| Carousels | No organic carousels (sponsored only) | Supported |
| Media mixing | Cannot mix images with videos or documents | Supported |

#### Telegram Bot API

| Restriction | Standard Bot API | Local Bot API Server |
|-------------|------------------|----------------------|
| File size limit | 50 MB | 2 GB |
| Caption length | 1,024 characters | 1,024 characters |
| Premium features | Not available to bots | Not available to bots |

#### Bluesky API

| Restriction | Details |
|-------------|---------|
| Daily limit | 25 videos per day (sourced from `@publora/platform-limits`) |
| Prerequisites | Email verification required before video uploads |

---

## Platform Requirements

Different platforms have different requirements for media. Some platforms are text-only capable, while others require media for every post.

<!-- synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->
| Platform | Requires Media | Requires Video | Supports Text-Only | Supports Threading |
|----------|----------------|----------------|--------------------|--------------------|
| **Twitter/X** | No | No | Yes | Yes |
| **Instagram** | Yes | No | No | No |
| **Threads** | No | No | Yes | No |
| **TikTok** | Yes | No | No | No |
| **LinkedIn** | No | No | Yes | No |
| **YouTube** | Yes | Yes | No | No |
| **Facebook** | No | No | Yes | No |
| **Mastodon** | No | No | Yes | No |
| **Bluesky** | No | No | Yes | No |
| **Telegram** | No | No | Yes | No |
| **Pinterest** | Yes | No | No | No — not currently supported |

### Media Requirement Notes

- **TikTok** requires media and supports image carousels or video; **YouTube** requires video. Neither supports text-only publication.
- **Instagram** requires at least one image or video with every post.
- **Pinterest** is listed for reference but is **not currently supported** in Publora. Support is planned for a future release.
- **Threading** is currently available for Twitter/X; multi-part Threads publishing is disabled.

---

## Rate Limits

Platform-side posting quotas are advisory, account-dependent, and may change without notice; they are not a Publora contract. Unsourced and conflicting posts-per-hour/day numbers are intentionally omitted.

Publora itself enforces plan entitlements and the media-URL ingestion limit (60 URLs per fixed one-hour window, returned as `429 MEDIA_URL_RATE_LIMITED` with `Retry-After`).

### Rate Limit Best Practices

1. **Space out posts** rather than bulk-posting at the same time.
2. **Implement exponential backoff** when you receive rate limit errors.
3. **Track your posting frequency** per platform to stay within limits.
4. **Treat platform failures explicitly**; do not assume automatic retry or redistribution.

---

## Common API Error Messages

When posts fail, the platform returns specific error messages. Understanding these errors helps you diagnose and fix issues quickly.

### Instagram Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `(#10) The user is not an Instagram Business` | Meta rejected the connected account for this operation | Reconnect an eligible professional account through Instagram Login for Business |
| `Error 2207010` | Caption limit exceeded | Reduce caption to under 2,200 characters |
| `Error 2207004` | Image exceeds 8 MB | Compress image to under 8 MB |
| `Error 9, Subcode 2207042` | Publishing rate limit reached | Wait before posting again |

### TikTok Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `spam_risk_too_many_posts` | Daily post limit reached | Wait 24 hours before posting again |
| `duration_check_failed` | Video must be 3s-10min | Adjust video duration |

### Twitter/X Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `This user is not allowed to post a video longer than 2 minutes` | X rejected the upload for its own duration/account rules; Publora's package limit is 140 seconds (2:20) | Keep the video at or below 140 seconds; if X still rejects it, surface the platform error without assuming an account tier |

### Facebook Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `Error 1363026` | Video exceeds 40 min duration | Trim video to under 45 minutes |
| `Error 1363023` | File size exceeds 2 GB | Compress video to under 2 GB |
| `Error 1363128` | Reels duration outside 3-90 second range | Adjust Reel to 3-90 seconds |

### LinkedIn Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `MEDIA_ASSET_PROCESSING_FAILED` | File too large or unsupported format | Check file size (<500 MB) and format (MP4) |
| `Error 429` | Rate limit exceeded | Wait and retry with backoff |

### Telegram Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `MEDIA_CAPTION_TOO_LONG` | Caption exceeds 1024 chars (bots) | Reduce caption length |
| `Bad Request: file is too big` | File exceeds 50 MB | Compress file to under 50 MB |

### Bluesky Errors

| Error | Meaning | Resolution |
|-------|---------|------------|
| `429 Too Many Requests` | Rate limit exceeded | Wait and retry |
| Video job state `JOB_STATE_FAILED` | Video processing failed | Check video format and size |

---

## Validation Error Codes

The Publora validation system returns standardized error codes when content does not meet platform requirements. Use these codes to provide clear feedback to users.

### Content Errors

| Error Code | Description |
|------------|-------------|
| `CONTENT_TOO_LONG` | Content exceeds the platform's character limit |
| `CONTENT_TOO_SHORT` | Content is below the minimum required length |
| `CONTENT_REQUIRED` | Text content is required for this platform |
| `CONTENT_OR_MEDIA_REQUIRED` | Either text content or media is required |

### Media Errors

| Error Code | Description |
|------------|-------------|
| `MEDIA_REQUIRED` | Platform requires at least one media file |
| `MEDIA_SIZE_EXCEEDED` | File size exceeds the platform limit |
| `MEDIA_COUNT_EXCEEDED` | Too many media files attached |
| `MEDIA_TYPE_NOT_SUPPORTED` | File format is not supported by the platform |
| `MEDIA_DIMENSIONS_INVALID` | Image dimensions are outside the allowed range |

### Video Errors

| Error Code | Description |
|------------|-------------|
| `VIDEO_REQUIRED` | Platform requires a video (YouTube) |
| `VIDEO_DURATION_EXCEEDED` | Video duration exceeds the platform limit |
| `VIDEO_DURATION_TOO_SHORT` | Video is shorter than the minimum required duration |
| `VIDEO_NOT_SUPPORTED` | Platform does not support video content |
| `IMAGES_NOT_SUPPORTED` | Platform does not accept images (for example, YouTube) |

### Platform Errors

| Error Code | Description |
|------------|-------------|
| `PLATFORM_NOT_SUPPORTED` | Unknown or unsupported platform identifier |
| `PLATFORM_SETTING_REQUIRED` | A required platform-specific setting is missing |
| `PLATFORM_SETTING_INVALID` | A platform-specific setting has an invalid value |

---

---

## Sources

Official platform documentation used to compile these limits:

- [Twitter/X Developer Community](https://devcommunity.x.com/t/unable-to-post-videos-longer-than-2-minutes-via-post-2-tweets/248440)
- [Instagram Graph API - Content Publishing](https://developers.facebook.com/docs/instagram-platform/content-publishing/)
- [TikTok Content Posting API](https://developers.tiktok.com/doc/content-posting-api-reference-direct-post)
- [LinkedIn Videos API](https://learn.microsoft.com/en-us/linkedin/marketing/community-management/shares/videos-api)
- [Facebook Graph API](https://developers.facebook.com/docs/graph-api/)
- [Threads API](https://developers.facebook.com/docs/threads/)
- [Bluesky AT Protocol Docs](https://docs.bsky.app/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Mastodon API Documentation](https://docs.joinmastodon.org/api/)
- [Pinterest API v5](https://developers.pinterest.com/docs/api/v5/)
- [YouTube Data API v3](https://developers.google.com/youtube/v3/)


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
