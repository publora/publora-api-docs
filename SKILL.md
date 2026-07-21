---
name: publora-api
description: Use this skill when scheduling, publishing, or managing social media posts across multiple platforms (Twitter/X, LinkedIn, Instagram, Threads, TikTok, YouTube, Facebook, Bluesky, Mastodon, Telegram) via the Publora REST API. Also use for retrieving LinkedIn analytics and managing workspace users.
---

# Publora API Skill

This skill provides complete documentation for the Publora social media scheduling API.

## When to Use This Skill

- Scheduling posts to social media platforms
- Creating, updating, or deleting social media posts
- Uploading media (images/videos) for posts
- Retrieving LinkedIn analytics (impressions, reactions, followers)
- Adding reactions and comments to LinkedIn posts
- Setting up webhooks for post status notifications
- Managing workspace team members
- Cross-platform posting workflows

## API Overview

**Base URL:** `https://api.publora.com/api/v1`

## Prerequisites (User Must Complete)

**Important:** You cannot programmatically create accounts or obtain API keys. The user must:

1. **Create account** at [publora.com](https://publora.com) (free tier available)
2. **Connect social accounts** via OAuth in the dashboard (**Channels** → Add Channel)
3. **Generate API key** at **API** in sidebar, then provide it to you

## Pricing

| Plan | Price | Posts/Month | Platforms |
|------|-------|-------------|-----------|
| Starter | Free | 15 | All 10 |
| Pro | $2.99/account | 100/account | All platforms |
| Premium | $5.99/account | 500/account | All platforms |

## Authentication

Publora uses **API keys** (not OAuth tokens). Keys never expire and don't require refresh.

**Header:** `x-publora-key: sk_YOUR_API_KEY`

```javascript
const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: { 'x-publora-key': 'sk_YOUR_API_KEY' }
});
```

**Note:** The MCP server uses `Authorization: Bearer sk_...` instead. Same key, different header format.

## Quick Reference

### Posts

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/list-posts` | GET | List posts with filters |
| `/create-post` | POST | Schedule/publish a post |
| `/get-post/:postGroupId` | GET | Get post details |
| `/update-post/:postGroupId` | PUT | Edit content, targets, timing, or status of a draft/scheduled post |
| `/delete-post/:postGroupId` | DELETE | Delete a post |
| `/get-upload-url` | POST | Get presigned URL for media |
| `/upload-instagram-cover` | POST | Upload a custom Instagram Reel cover (multipart) |
| `/upload-youtube-thumbnail` | POST | Upload a custom YouTube thumbnail (multipart) |
| `/platform-limits` | GET | Get live per-platform limits (no params) |
| `/post-logs/:postGroupId` | GET | Get publishing logs for a post |

### Connections

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/platform-connections` | GET | List connected accounts |
| `/test-connection/:platformId` | POST | Test if a connection is valid |

### LinkedIn Analytics & Engagement

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/linkedin-post-statistics` | POST | Get post metrics |
| `/linkedin-account-statistics` | POST | Get account metrics |
| `/linkedin-followers` | POST | Get follower count/growth |
| `/linkedin-profile-summary` | POST | Get profile overview |
| `/linkedin-reactions` | POST | Add reaction to a post |
| `/linkedin-reactions` | DELETE | Remove reaction from a post |
| `/linkedin-comments` | POST | Post a comment |
| `/linkedin-comments` | DELETE | Delete a comment |
| `/linkedin-reshare` | POST | Reshare an existing post |

### Webhooks

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/webhooks` | GET | List all webhooks |
| `/webhooks` | POST | Create a webhook |
| `/webhooks/:id` | PATCH | Update a webhook |
| `/webhooks/:id` | DELETE | Delete a webhook |
| `/webhooks/:id/regenerate-secret` | POST | Regenerate webhook secret |

### Workspace

> **Note:** Workspace access requires approval. Contact support@publora.com to enable.

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/workspace/users` | GET | List team members |
| `/workspace/users` | POST | Add team member |
| `/workspace/users/attach` | POST | Attach an existing Publora user (needs the user's consent token) |
| `/workspace/users/:userId` | DELETE | Remove team member |

## Common Patterns

### Schedule a Post

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'sk_YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Hello world!',
    platforms: ['linkedin-ABC123', 'twitter-456'],
    scheduledTime: '2026-03-01T14:00:00.000Z'
  })
});
```

### Get Connected Platforms

```javascript
const response = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers: { 'x-publora-key': 'sk_YOUR_API_KEY' }
});
const { connections } = await response.json();
// Use connections[].platformId for posting
```

### Get LinkedIn Analytics

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'sk_YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789',
    platformId: 'linkedin-ABC123',
    queryTypes: 'ALL'
  })
});
```

### Comment on LinkedIn Post

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'sk_YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:ugcPost:7123456789',
    platformId: 'linkedin-ABC123',
    message: 'Great insights! Thanks for sharing.'
  })
});
```

### Create Webhook

```javascript
const response = await fetch('https://api.publora.com/api/v1/webhooks', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'sk_YOUR_API_KEY'
  },
  body: JSON.stringify({
    name: 'Production Notifications',
    url: 'https://your-server.com/webhook',
    events: ['post.published', 'post.failed']
  })
});
```

## Key Concepts

### Platform IDs

Platform IDs follow the format `platform-id`:
- `twitter-123456789`
- `linkedin-Tz9W5i6ZYG`
- `instagram-17841412345678`

Get platform IDs from the `/platform-connections` endpoint.

### Scheduled Time Format

Use ISO 8601 UTC format:
```
2026-03-15T14:00:00.000Z  // Correct
2026-03-15T14:00:00Z      // Also correct
```

Time must be in the future. A past time is never silently accepted:

- **Less than 5 min in the past** (clock skew): clamped to server time, and the `200` response carries `warnings: [{ code: "SCHEDULED_TIME_COERCED", requested, effective }]`. This tolerance is permanent.
- **5 min or more in the past**: clamped + warned today; scheduled to become `400 SCHEDULED_TIME_IN_PAST` on **2026-08-25**, unless production configuration overrides the date either way.

A `200` can still mean the API adjusted your request — read `warnings`, don't discard it. `GET /get-post` returns the effective stored `scheduledTime` to confirm what was actually kept.

### Retry Safety (Idempotency-Key)

Send an optional `Idempotency-Key` header on `create-post` / `update-post`. A timeout never tells you whether the request landed — without a key, the retry **creates a second post** (and on `update-post` with `mediaUrls`, appends the media twice).

```javascript
const key = crypto.randomUUID();  // one key per logical operation — reuse it for every retry
await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'sk_YOUR_API_KEY',
    'Idempotency-Key': key
  },
  body: JSON.stringify({ content: 'Hello', platforms: ['twitter-123456789'] })
});
```

- Same key + same body → the original response is replayed (no second post).
- Same key + different body → `422` / `IDEMPOTENCY_KEY_CONFLICT`.
- Same key still processing → `409` / `IDEMPOTENCY_IN_FLIGHT` — wait and retry the identical call, don't switch keys.
- Keys are per-user and expire after 24h. Opt-in: without the header, behaviour is unchanged.

### Post Statuses

- `draft` - Not scheduled
- `scheduled` - Waiting to publish
- `published` - Successfully posted
- `failed` - Publishing failed
- `partially_published` - Some platforms failed

### LinkedIn Query Types

Available metrics: `IMPRESSION`, `MEMBERS_REACHED`, `RESHARE`, `REACTION`, `COMMENT`

Use `queryTypes: 'ALL'` to get all metrics at once.

### Webhook Events

Available events:
- `post.scheduled` - Post was scheduled
- `post.published` - Post was successfully published
- `post.failed` - Post failed to publish
- `post.demoted` - Scheduled post returned to draft after media attach/detach
- `token.expiring` - Defined and subscribable, but not currently dispatched; do not depend on it

## Error Handling

Always check response status:

```javascript
if (!response.ok) {
  const error = await response.json();
  console.error(error.error); // Error message
}
```

Common errors:
- 401: Invalid API key
- 400: Invalid parameters
- 404: Resource not found

Machine-readable `code` values (branch on `code`, not on the message text):
- `SCHEDULED_TIME_IN_PAST` (400) — `scheduledTime` 5+ min in the past; body carries `serverTime`
- `PLATFORM_SETTING_UNKNOWN` (400) — unknown `platformSettings` path; body carries the exact `field`
- `IDEMPOTENCY_KEY_CONFLICT` (422) — key reused with a different body
- `IDEMPOTENCY_IN_FLIGHT` (409) — same key still processing; retry the identical call

Successful responses may carry a `warnings` array (e.g. `SCHEDULED_TIME_COERCED`) — surface it. Full reference: [Error Handling](docs/guides/error-handling.md).

## Resources

<!-- Maintainers: the backend /api/v1/x-bookmarks and /auth/x-bookmarks routes are internal (dashboard X-bookmark sync, gated by requireXBookmarksAccess) — intentionally undocumented, do not add to public API docs. -->

See the `docs/` directory for:
- `docs/endpoints/` - Detailed endpoint documentation
- `docs/guides/` - How-to guides
- `docs/examples/` - Code examples in multiple languages
- `docs/platforms/` - Platform-specific information
