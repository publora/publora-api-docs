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
- Managing workspace team members
- Cross-platform posting workflows

## API Overview

**Base URL:** `https://api.publora.com/api/v1`

**Authentication:** Header `x-publora-key: sk_your_api_key`

## Quick Reference

### Posts

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/list-posts` | GET | List posts with filters |
| `/create-post` | POST | Schedule/publish a post |
| `/get-post/:postGroupId` | GET | Get post details |
| `/update-post/:postGroupId` | PUT | Reschedule or change status |
| `/delete-post/:postGroupId` | DELETE | Delete a post |
| `/get-upload-url` | POST | Get presigned URL for media |

### Connections

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/platform-connections` | GET | List connected accounts |

### LinkedIn Analytics

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/linkedin-post-statistics` | POST | Get post metrics |
| `/linkedin-account-statistics` | POST | Get account metrics |
| `/linkedin-followers` | POST | Get follower count/growth |
| `/linkedin-profile-summary` | POST | Get profile overview |
| `/linkedin-create-reaction` | POST | React to a post |
| `/linkedin-delete-reaction` | DELETE | Remove reaction |

### Workspace

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/workspace-users` | GET | List team members |
| `/workspace-users` | POST | Add team member |
| `/workspace-users/:userId` | DELETE | Remove team member |

## Common Patterns

### Schedule a Post

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'sk_your_api_key'
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
  headers: { 'x-publora-key': 'sk_your_api_key' }
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
    'x-publora-key': 'sk_your_api_key'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789',
    platformId: 'linkedin-ABC123',
    queryTypes: 'ALL'
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

Time must be in the future.

### Post Statuses

- `draft` - Not scheduled
- `scheduled` - Waiting to publish
- `published` - Successfully posted
- `failed` - Publishing failed
- `partially_published` - Some platforms failed

### LinkedIn Query Types

Available metrics: `IMPRESSION`, `MEMBERS_REACHED`, `RESHARE`, `REACTION`, `COMMENT`

Use `queryTypes: 'ALL'` to get all metrics at once.

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

## Resources

See the `docs/` directory for:
- `docs/endpoints/` - Detailed endpoint documentation
- `docs/guides/` - How-to guides
- `docs/examples/` - Code examples in multiple languages
- `docs/platforms/` - Platform-specific information
