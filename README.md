# Publora API Documentation

**Affordable REST API for scheduling and publishing social media posts across 10 platforms.**

Schedule posts to X/Twitter, LinkedIn, Instagram, Threads, TikTok, YouTube, Facebook, Bluesky, Mastodon, and Telegram — all from a single API call. Starting at **$9/month** with full API access. 14-day free trial, no credit card needed.

**Website:** [publora.com](https://publora.com) | **Dashboard:** [app.publora.com](https://app.publora.com) | **Email:** sev@publora.com

## Quick Start

```bash
# 1. List your connected social accounts
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY"

# 2. Schedule a post
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Hello from Publora API!",
    "platforms": ["twitter-123456789", "linkedin-ABC123"],
    "scheduledTime": "2026-03-01T14:00:00.000Z"
  }'
```

**3 API calls. 10 platforms. $9/month.**

## API Endpoints

| Method | Endpoint | Description | Docs |
|--------|----------|-------------|------|
| `GET` | `/platform-connections` | List connected social accounts | [View](docs/endpoints/platform-connections.md) |
| `POST` | `/create-post` | Create and schedule a post | [View](docs/endpoints/create-post.md) |
| `GET` | `/get-post/:postGroupId` | Get post details and status | [View](docs/endpoints/get-post.md) |
| `PUT` | `/update-post/:postGroupId` | Update post timing or status | [View](docs/endpoints/update-post.md) |
| `DELETE` | `/delete-post/:postGroupId` | Delete a scheduled post | [View](docs/endpoints/delete-post.md) |
| `POST` | `/get-upload-url` | Get pre-signed URL for media upload | [View](docs/endpoints/upload-media.md) |
| `POST` | `/linkedin-post-statistics` | Get LinkedIn post analytics | [View](docs/endpoints/linkedin-statistics.md) |
| `POST` | `/linkedin-account-statistics` | Get LinkedIn account analytics | [View](docs/endpoints/linkedin-statistics.md) |
| `POST` | `/linkedin-reactions` | Add reaction to a LinkedIn post | [View](docs/endpoints/linkedin-reactions.md) |
| `DELETE` | `/linkedin-reactions` | Remove a LinkedIn reaction | [View](docs/endpoints/linkedin-reactions.md) |

Base URL: `https://api.publora.com/api/v1`

## Supported Platforms

| Platform | Text | Images | Videos | Threading | Analytics |
|----------|------|--------|--------|-----------|-----------|
| X / Twitter | 280 chars | Up to 4 | 1 per post | Auto-split | — |
| LinkedIn | 3,000 chars | Multiple | 1 per post | — | 5 metrics |
| Instagram | 2,200 chars | Carousel (10) | Reels/Stories | — | — |
| Threads | 500 chars | Carousel | 1 per post | Auto-split | — |
| TikTok | Caption | — | 1 per post (MP4) | — | — |
| YouTube | Description | — | 1 per post | — | — |
| Facebook | 63,206 chars | Multiple | 1 per post | — | — |
| Bluesky | 300 chars | Up to 4 | 1 per post | — | — |
| Mastodon | 500 chars | Up to 4 | 1 per post | — | — |
| Telegram | 4,096 chars | Multiple | 1 per post | — | — |

## Authentication

All requests require the `x-publora-key` header:

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: sk_1234567890.abcdef1234567890"
```

Get your API key: [publora.com](https://publora.com) → Settings → API Keys → Generate.

See [Authentication Guide](docs/authentication.md) for details.

## Pricing

| Plan | Monthly | Yearly | Social Accounts | Video Upload |
|------|---------|--------|-----------------|--------------|
| Starter | $12/mo | $9/mo | 10 | 100 MB |
| Pro | $19/mo | $15/mo | 30 | 250 MB |
| Premium | $35/mo | $25/mo | Unlimited | Custom |

All plans include full API access. [Start free trial](https://publora.com).

## Documentation

### Getting Started
- [Quick Start Guide](docs/getting-started.md) — first post in 60 seconds
- [Authentication](docs/authentication.md) — API keys and workspace auth

### Endpoint Reference
- [Create Post](docs/endpoints/create-post.md) — schedule posts across platforms
- [Get Post](docs/endpoints/get-post.md) — check post status and details
- [Update Post](docs/endpoints/update-post.md) — reschedule or change status
- [Delete Post](docs/endpoints/delete-post.md) — remove posts and media
- [Platform Connections](docs/endpoints/platform-connections.md) — list connected accounts
- [Upload Media](docs/endpoints/upload-media.md) — images and video uploads
- [LinkedIn Statistics](docs/endpoints/linkedin-statistics.md) — post and account analytics
- [LinkedIn Reactions](docs/endpoints/linkedin-reactions.md) — add/remove reactions

### Platform Guides
- [X / Twitter](docs/platforms/x-twitter.md) · [LinkedIn](docs/platforms/linkedin.md) · [Instagram](docs/platforms/instagram.md) · [Threads](docs/platforms/threads.md) · [TikTok](docs/platforms/tiktok.md) · [YouTube](docs/platforms/youtube.md) · [Facebook](docs/platforms/facebook.md) · [Bluesky](docs/platforms/bluesky.md) · [Mastodon](docs/platforms/mastodon.md) · [Telegram](docs/platforms/telegram.md)

### Usage Guides
- [Scheduling Posts](docs/guides/scheduling.md) — timing, drafts, batch scheduling
- [Media Uploads](docs/guides/media-uploads.md) — images, videos, carousels
- [Cross-Platform Posting](docs/guides/cross-platform.md) — one call, many platforms
- [Error Handling](docs/guides/error-handling.md) — status codes, retries
- [Workspace / B2B API](docs/guides/workspace.md) — managed users, white-label

### API Specification
- [OpenAPI 3.0 Spec](schema/openapi.yaml)

## About

**[Publora](https://publora.com)** is an affordable social media API built by **[Creative Content Crafts, Inc.](https://cccrafts.ai)**

Looking for AI-powered content creation for LinkedIn, Threads, and X? Check out **[Co.Actor](https://co.actor)** — our AI service that helps B2B teams create authentic thought leadership content at scale.

- **Publora** ([publora.com](https://publora.com)) — schedule and publish posts via API
- **Co.Actor** ([co.actor](https://co.actor)) — AI content creation for LinkedIn, Threads, and X
- **Creative Content Crafts** ([cccrafts.ai](https://cccrafts.ai)) — the company behind it all

## License

[MIT](LICENSE)
