# Publora API Documentation

**Affordable REST API for scheduling and publishing social media posts across 10 platforms.**

Schedule posts to X/Twitter, LinkedIn, Instagram, Threads, TikTok, YouTube, Facebook, Bluesky, Mastodon, and Telegram — all from a single API call. **Free tier available**, paid plans from **$2.99/month** per connected account.

**Website:** [publora.com](https://publora.com) | **Dashboard:** [app.publora.com](https://app.publora.com) | **Email:** serge@publora.com

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

**3 API calls. 10 platforms. Free tier + $2.99/account.**

## Why Publora?

### Price Comparison

| Feature | Publora | Ayrshare | Publer | Sprout Social |
|---------|---------|----------|--------|---------------|
| **Starting price** | **Free** | $49/mo | $12/mo | $249/mo |
| Per-account pricing | $2.99-5.99 | N/A | N/A | N/A |
| Platforms | **10** | 13 | 9 | 6 |
| API access | All plans | Paid only | Paid only | Enterprise |
| Bluesky support | Yes | Yes | No | No |
| Threads support | Yes | Yes | Yes | No |
| Mastodon support | Yes | No | Yes | No |

**Publora offers a free tier and flexible per-account pricing — pay only for what you use.**

### Why Developers Choose Publora

1. **Affordable** — Free tier available, paid plans from $2.99/account. No enterprise tier required.
2. **10 Platforms** — X, LinkedIn, Instagram, Threads, TikTok, YouTube, Facebook, Bluesky, Mastodon, Telegram.
3. **API-First** — Clean REST API designed for developers, not a bloated dashboard.
4. **AI-Ready** — Docs indexed on [Context7](https://context7.com) so AI coding assistants already know our API.
5. **Modern Platforms** — First-class support for Bluesky, Threads, and Mastodon that competitors lack.

## API Endpoints

| Method | Endpoint | Description | Docs |
|--------|----------|-------------|------|
| `GET` | `/platform-connections` | List connected social accounts | [View](docs/endpoints/platform-connections.md) |
| `POST` | `/test-connection/:platformId` | Test a platform connection | [View](docs/endpoints/test-connection.md) |
| `POST` | `/create-post` | Create and schedule a post | [View](docs/endpoints/create-post.md) |
| `GET` | `/list-posts` | List all posts with pagination | [View](docs/endpoints/list-posts.md) |
| `GET` | `/get-post/:postGroupId` | Get post details and status | [View](docs/endpoints/get-post.md) |
| `GET` | `/post-logs/:postGroupId` | Get publish attempt history | [View](docs/endpoints/post-logs.md) |
| `PUT` | `/update-post/:postGroupId` | Update post timing or status | [View](docs/endpoints/update-post.md) |
| `DELETE` | `/delete-post/:postGroupId` | Delete a scheduled post | [View](docs/endpoints/delete-post.md) |
| `POST` | `/get-upload-url` | Get pre-signed URL for media upload | [View](docs/endpoints/upload-media.md) |
| `POST` | `/upload-instagram-cover` | Upload a custom Instagram Reel cover | [View](docs/endpoints/upload-instagram-cover.md) |
| `GET` | `/platform-limits` | Get live per-platform limits | [View](docs/endpoints/platform-limits.md) |
| `POST` | `/upload-youtube-thumbnail` | Upload a custom YouTube thumbnail | [View](docs/endpoints/upload-youtube-thumbnail.md) |
| `GET/POST` | `/webhooks` | Manage webhook notifications | [View](docs/endpoints/webhooks.md) |
| `POST` | `/linkedin-post-statistics` | Get LinkedIn post analytics | [View](docs/endpoints/linkedin-statistics.md) |
| `POST` | `/linkedin-account-statistics` | Get LinkedIn account analytics | [View](docs/endpoints/linkedin-statistics.md) |
| `POST` | `/linkedin-reactions` | Add reaction to a LinkedIn post | [View](docs/endpoints/linkedin-reactions.md) |
| `DELETE` | `/linkedin-reactions` | Remove a LinkedIn reaction | [View](docs/endpoints/linkedin-reactions.md) |
| `POST` | `/linkedin-reshare` | Reshare an existing LinkedIn post | [View](docs/endpoints/linkedin-reshare.md) |
| `POST` | `/linkedin-followers` | Get LinkedIn follower statistics | [View](docs/endpoints/linkedin-followers.md) |
| `POST` | `/linkedin-profile-summary` | Get LinkedIn profile summary | [View](docs/endpoints/linkedin-profile-summary.md) |

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
  -H "x-publora-key: sk_YOUR_API_KEY"
```

Get your API key: [publora.com](https://publora.com) → **API** in sidebar → Generate.

See [Authentication Guide](docs/authentication.md) for details.

## Pricing

| Plan | Price | Posts/Month | Accounts | Platforms | Video |
|------|-------|-------------|----------|-----------|-------|
| **Starter** | Free | 15 | 3 | All 10 | 50 MB |
| **Pro** | $2.99/account | 100/account | Unlimited | All 10 | 100 MB |
| **Premium** | $5.99/account | 500/account | Unlimited | All 10 | 250 MB |

All plans include full API access. Pro/Premium use per-account pricing — add as many accounts as you need. [Get started free](https://publora.com).

## Documentation

### Getting Started
- [Quick Start Guide](docs/getting-started.md) — first post in 60 seconds
- [Authentication](docs/authentication.md) — API keys and workspace auth

### Endpoint Reference
- [Create Post](docs/endpoints/create-post.md) — schedule posts across platforms
- [List Posts](docs/endpoints/list-posts.md) — fetch all posts with pagination and filters
- [Get Post](docs/endpoints/get-post.md) — check post status and error details
- [Post Logs](docs/endpoints/post-logs.md) — publish attempt history for debugging
- [Update Post](docs/endpoints/update-post.md) — reschedule or change status
- [Delete Post](docs/endpoints/delete-post.md) — remove posts across all platforms
- [Platform Connections](docs/endpoints/platform-connections.md) — list connected accounts with health status
- [Test Connection](docs/endpoints/test-connection.md) — validate a connection before posting
- [Webhooks](docs/endpoints/webhooks.md) — real-time notifications for post events
- [Upload Media](docs/endpoints/upload-media.md) — images and video uploads
- [Upload Instagram Cover](docs/endpoints/upload-instagram-cover.md) — custom cover image for Reels
- [Upload YouTube Thumbnail](docs/endpoints/upload-youtube-thumbnail.md) — custom video thumbnail (two-step: upload → update-post)
- [Platform Limits](docs/endpoints/platform-limits.md) — live per-platform character/media limits as JSON
- [LinkedIn Statistics](docs/endpoints/linkedin-statistics.md) — post and account analytics
- [LinkedIn Reactions](docs/endpoints/linkedin-reactions.md) — add/remove reactions
- [LinkedIn Reshare](docs/endpoints/linkedin-reshare.md) — repost an existing LinkedIn post

### Platform Guides
- [X / Twitter](docs/platforms/x-twitter.md) · [LinkedIn](docs/platforms/linkedin.md) · [Instagram](docs/platforms/instagram.md) · [Threads](docs/platforms/threads.md) · [TikTok](docs/platforms/tiktok.md) · [YouTube](docs/platforms/youtube.md) · [Facebook](docs/platforms/facebook.md) · [Bluesky](docs/platforms/bluesky.md) · [Mastodon](docs/platforms/mastodon.md) · [Telegram](docs/platforms/telegram.md)

### Usage Guides
- [Scheduling Posts](docs/guides/scheduling.md) — timing, drafts, batch scheduling
- [Bulk Scheduling](docs/guides/bulk-scheduling.md) — CSV import, weekly content batches
- [Threading Guide](docs/guides/threading.md) — post multi-part threads via API
- [Twitter Threads](docs/guides/twitter-threads.md) — tweet thread automation
- [Threads Multi-Post](docs/guides/threads-multi-post.md) — Meta Threads threading
- [Rate Limits & Optimal Times](docs/guides/rate-limits.md) — platform limits, peak engagement, queue scheduling
- [Media Uploads](docs/guides/media-uploads.md) — images, videos, carousels
- [Cross-Platform Posting](docs/guides/cross-platform.md) — one call, many platforms
- [LinkedIn Analytics](docs/guides/analytics.md) — post performance, account metrics
- [Error Handling](docs/guides/error-handling.md) — status codes, retries
- [Workspace / B2B API](docs/guides/workspace.md) — managed users, white-label

### AI Integration
- [MCP Server](docs/guides/mcp-server.md) — Claude Code, Claude Desktop, Cursor integration
- [Cursor AI Guide](docs/guides/cursor-ai.md) — AI-assisted development with Publora

### Code Examples
- [JavaScript Examples](docs/examples/javascript/) — fetch, axios, Node.js
- [Python Examples](docs/examples/python/) — requests, async workflows
- [cURL Examples](docs/examples/curl/) — command-line reference
- [Zapier Integration](docs/examples/no-code/zapier-integration.md) — no-code automation
- [Make Integration](docs/examples/no-code/make-integration.md) — visual workflows

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
