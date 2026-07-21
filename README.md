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
| `GET` | `/platform-connections` | List connected social accounts | [View](https://docs.publora.com/endpoints/platform-connections) |
| `POST` | `/test-connection/:platformId` | Test a platform connection | [View](https://docs.publora.com/endpoints/test-connection) |
| `POST` | `/create-post` | Create and schedule a post | [View](https://docs.publora.com/endpoints/create-post) |
| `GET` | `/list-posts` | List all posts with pagination | [View](https://docs.publora.com/endpoints/list-posts) |
| `GET` | `/get-post/:postGroupId` | Get post details and status | [View](https://docs.publora.com/endpoints/get-post) |
| `GET` | `/post-logs/:postGroupId` | Get publish attempt history | [View](https://docs.publora.com/endpoints/post-logs) |
| `PUT` | `/update-post/:postGroupId` | Edit a draft/scheduled post: content, targets, timing, or status | [View](https://docs.publora.com/endpoints/update-post) |
| `DELETE` | `/delete-post/:postGroupId` | Delete a scheduled post | [View](https://docs.publora.com/endpoints/delete-post) |
| `POST` | `/get-upload-url` | Get pre-signed URL for media upload | [View](https://docs.publora.com/endpoints/upload-media) |
| `POST` | `/upload-instagram-cover` | Upload a custom Instagram Reel cover | [View](https://docs.publora.com/endpoints/upload-instagram-cover) |
| `GET` | `/platform-limits` | Get live per-platform limits | [View](https://docs.publora.com/endpoints/platform-limits) |
| `POST` | `/upload-youtube-thumbnail` | Upload a custom YouTube thumbnail | [View](https://docs.publora.com/endpoints/upload-youtube-thumbnail) |
| `GET/POST` | `/webhooks` | Manage webhook notifications | [View](https://docs.publora.com/endpoints/webhooks) |
| `POST` | `/linkedin-post-statistics` | Get LinkedIn post analytics | [View](https://docs.publora.com/endpoints/linkedin-statistics) |
| `POST` | `/linkedin-account-statistics` | Get LinkedIn account analytics | [View](https://docs.publora.com/endpoints/linkedin-statistics) |
| `POST` | `/linkedin-reactions` | Add reaction to a LinkedIn post | [View](https://docs.publora.com/endpoints/linkedin-reactions) |
| `DELETE` | `/linkedin-reactions` | Remove a LinkedIn reaction | [View](https://docs.publora.com/endpoints/linkedin-reactions) |
| `POST` | `/linkedin-reshare` | Reshare an existing LinkedIn post | [View](https://docs.publora.com/endpoints/linkedin-reshare) |
| `POST` | `/linkedin-followers` | Get LinkedIn follower statistics | [View](https://docs.publora.com/endpoints/linkedin-followers) |
| `POST` | `/linkedin-profile-summary` | Get LinkedIn profile summary | [View](https://docs.publora.com/endpoints/linkedin-profile-summary) |

Base URL: `https://api.publora.com/api/v1`

## Supported Platforms

| Platform | Text | Images | Videos | Threading | Analytics |
|----------|------|--------|--------|-----------|-----------|
| X / Twitter | 280 chars | Up to 4 | 1 per post | Auto-split | — |
| LinkedIn | 3,000 chars | Multiple | 1 per post | — | 5 metrics |
| Instagram | 2,200 chars | Carousel (10) | Reels/Stories | — | — |
| Threads | 500 chars | Carousel | 1 per post | Disabled | — |
| TikTok | Caption | Up to 35 | 1 per post (MP4/MOV/WebM) | — | — |
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

See [Authentication Guide](https://docs.publora.com/authentication) for details.

## Pricing

| Plan | Price | Posts/Month | Accounts | Platforms | Video |
|------|-------|-------------|----------|-----------|-------|
| **Starter** | Free | 15 | 3 | All 10 | 50 MB |
| **Pro** | $2.99/account | 100/account | Unlimited | All 10 | 100 MB |
| **Premium** | $5.99/account | 500/account | Unlimited | All 10 | 250 MB |

All plans include full API access. Pro/Premium use per-account pricing — add as many accounts as you need. [Get started free](https://publora.com).

## Documentation

### Getting Started
- [Quick Start Guide](https://docs.publora.com/getting-started) — first post in 60 seconds
- [Authentication](https://docs.publora.com/authentication) — API keys and workspace auth

### Endpoint Reference
- [Create Post](https://docs.publora.com/endpoints/create-post) — schedule posts across platforms
- [List Posts](https://docs.publora.com/endpoints/list-posts) — fetch all posts with pagination and filters
- [Get Post](https://docs.publora.com/endpoints/get-post) — check post status and error details
- [Post Logs](https://docs.publora.com/endpoints/post-logs) — publish attempt history for debugging
- [Update Post](https://docs.publora.com/endpoints/update-post) — edit content, targets, timing, or status before publication
- [Delete Post](https://docs.publora.com/endpoints/delete-post) — remove posts across all platforms
- [Platform Connections](https://docs.publora.com/endpoints/platform-connections) — list connected accounts with health status
- [Test Connection](https://docs.publora.com/endpoints/test-connection) — validate a connection before posting
- [Webhooks](https://docs.publora.com/endpoints/webhooks) — real-time notifications for post events
- [Upload Media](https://docs.publora.com/endpoints/upload-media) — images and video uploads
- [Upload Instagram Cover](https://docs.publora.com/endpoints/upload-instagram-cover) — custom cover image for Reels
- [Upload YouTube Thumbnail](https://docs.publora.com/endpoints/upload-youtube-thumbnail) — custom video thumbnail (two-step: upload → update-post)
- [Platform Limits](https://docs.publora.com/endpoints/platform-limits) — live per-platform character/media limits as JSON
- [LinkedIn Statistics](https://docs.publora.com/endpoints/linkedin-statistics) — post and account analytics
- [LinkedIn Reactions](https://docs.publora.com/endpoints/linkedin-reactions) — add/remove reactions
- [LinkedIn Reshare](https://docs.publora.com/endpoints/linkedin-reshare) — repost an existing LinkedIn post

### Platform Guides
- [X / Twitter](https://docs.publora.com/platforms/x-twitter) · [LinkedIn](https://docs.publora.com/platforms/linkedin) · [Instagram](https://docs.publora.com/platforms/instagram) · [Threads](https://docs.publora.com/platforms/threads) · [TikTok](https://docs.publora.com/platforms/tiktok) · [YouTube](https://docs.publora.com/platforms/youtube) · [Facebook](https://docs.publora.com/platforms/facebook) · [Bluesky](https://docs.publora.com/platforms/bluesky) · [Mastodon](https://docs.publora.com/platforms/mastodon) · [Telegram](https://docs.publora.com/platforms/telegram)

### Usage Guides
- [Scheduling Posts](https://docs.publora.com/guides/scheduling) — timing, drafts, batch scheduling
- [Bulk Scheduling](https://docs.publora.com/guides/bulk-scheduling) — CSV import, weekly content batches
- [Threading Guide](https://docs.publora.com/guides/threading) — post multi-part threads via API
- [Twitter Threads](https://docs.publora.com/guides/twitter-threads) — tweet thread automation
- [Threads Multi-Post](https://docs.publora.com/guides/threads-multi-post) — Meta Threads threading
- [Rate Limits & Optimal Times](https://docs.publora.com/guides/rate-limits) — platform limits, peak engagement, queue scheduling
- [Media Uploads](https://docs.publora.com/guides/media-uploads) — images, videos, carousels
- [Cross-Platform Posting](https://docs.publora.com/guides/cross-platform) — one call, many platforms
- [LinkedIn Analytics](https://docs.publora.com/guides/analytics) — post performance, account metrics
- [Error Handling](https://docs.publora.com/guides/error-handling) — status codes, retries
- [Workspace / B2B API](https://docs.publora.com/guides/workspace) — managed users, white-label

### AI Integration
- [MCP Server](https://docs.publora.com/guides/mcp-server) — Claude Code, Claude Desktop, Cursor integration
- [Cursor AI Guide](https://docs.publora.com/guides/cursor-ai) — AI-assisted development with Publora

### Code Examples
- [JavaScript Examples](https://docs.publora.com/examples/javascript/quick-start) — fetch, axios, Node.js
- [Python Examples](https://docs.publora.com/examples/python/quick-start) — requests, async workflows
- [cURL Examples](https://docs.publora.com/examples/curl/all-endpoints) — command-line reference
- [Zapier Integration](https://docs.publora.com/examples/no-code/zapier-integration) — no-code automation
- [Make Integration](https://docs.publora.com/examples/no-code/make-integration) — visual workflows

### API Specification
- [OpenAPI 3.0 Spec](schema/openapi.yaml)

### Public Repository Hygiene

Keep this public, Context7-indexed repository limited to user-facing documentation, `SKILL.md`, and published schemas. Internal audit reports, comparison scripts, private-host instructions, and temporary migration notes belong in the private Publora product repository, not here.

## About

**[Publora](https://publora.com)** is an affordable social media API built by **[Creative Content Crafts, Inc.](https://cccrafts.ai)**

Looking for AI-powered content creation for LinkedIn, Threads, and X? Check out **[Co.Actor](https://co.actor)** — our AI service that helps B2B teams create authentic thought leadership content at scale.

- **Publora** ([publora.com](https://publora.com)) — schedule and publish posts via API
- **Co.Actor** ([co.actor](https://co.actor)) — AI content creation for LinkedIn, Threads, and X
- **Creative Content Crafts** ([cccrafts.ai](https://cccrafts.ai)) — the company behind it all

## License

[MIT](https://github.com/publora/publora-api-docs/blob/main/LICENSE)
