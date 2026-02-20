# MCP Tools Reference

Complete reference for all 16 Publora MCP tools with parameters, examples, and code snippets.

## Posts Tools

### list_posts

List posts with optional filters for status, platform, and date range.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `status` | string | No | Filter by status: `draft`, `scheduled`, `published`, `failed`, `partially_published` |
| `platform` | string | No | Filter by platform: `twitter`, `linkedin`, `instagram`, `threads`, `tiktok`, `youtube`, `facebook`, `bluesky`, `mastodon`, `telegram` |
| `fromDate` | string | No | Start date (ISO 8601): `2026-02-01T00:00:00Z` |
| `toDate` | string | No | End date (ISO 8601): `2026-02-28T23:59:59Z` |
| `page` | number | No | Page number (default: 1) |
| `limit` | number | No | Results per page (default: 20, max: 100) |
| `sortBy` | string | No | Sort field: `createdAt`, `updatedAt`, `scheduledTime` (default: createdAt) |
| `sortOrder` | string | No | Sort direction: `asc` or `desc` (default: desc) |

**Example prompts:**

```text
"Show my scheduled posts"
"List all posts from last week"
"What LinkedIn posts are scheduled for next month?"
"Show me failed posts"
"List my drafts"
```

**Python example:**

```python
import asyncio
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def list_scheduled_posts():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # List scheduled posts
            result = await session.call_tool("list_posts", {
                "status": "scheduled",
                "limit": 50
            })
            print(result.content[0].text)

asyncio.run(list_scheduled_posts())
```

**Response example:**

```json
{
  "posts": [
    {
      "id": "pg_abc123",
      "content": "Excited to share our latest update!",
      "status": "scheduled",
      "scheduledTime": "2026-02-20T14:00:00Z",
      "platforms": ["linkedin-123456"]
    }
  ],
  "total": 1,
  "page": 1,
  "limit": 20
}
```

---

### create_post

Create and schedule a post to one or more platforms.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `content` | string | Yes | Post text content |
| `platforms` | string[] | Yes | Array of platform connection IDs (from `list_connections`) |
| `scheduledTime` | string | Yes | When to publish (ISO 8601): `2026-03-01T14:00:00Z` |

**Example prompts:**

```text
"Schedule 'Hello world!' to LinkedIn for tomorrow at 9am"
"Post 'We're hiring!' to Twitter and LinkedIn right now"
"Create a draft post for Instagram"
"Schedule this announcement to all my accounts for Monday"
```

**Python example:**

```python
import asyncio
from datetime import datetime, timedelta
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def schedule_post():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # First get connections to find platform IDs
            connections = await session.call_tool("list_connections", {})

            # Schedule a post for tomorrow at 9am UTC
            tomorrow_9am = (datetime.utcnow() + timedelta(days=1)).replace(
                hour=9, minute=0, second=0, microsecond=0
            ).isoformat() + "Z"

            result = await session.call_tool("create_post", {
                "content": "Excited to share our latest product update!",
                "platforms": ["linkedin-abc123"],
                "scheduledTime": tomorrow_9am
            })
            print(result.content[0].text)

asyncio.run(schedule_post())
```

**Platform limits:**

| Platform | Characters | Images | Video | Special Features |
|----------|------------|--------|-------|------------------|
| LinkedIn | 3,000 | 20 | 200MB | Documents, carousels |
| X/Twitter | 280 (25K premium) | 4 | 140s | Auto-threading |
| Instagram | 2,200 | 10 | 90s | Reels supported |
| Threads | 500 | 10 | 5min | Auto-threading |
| TikTok | 2,200 | N/A | 10min | Video-only platform |
| YouTube | 5,000 desc | N/A | 12h | Shorts support |
| Facebook | 63,206 | 10 | 240min | Page posts, Reels |
| Bluesky | 300 | 4 | N/A | Auto-facet detection |
| Mastodon | 500* | 4 | 40MB | Instance-variable |
| Telegram | 4,096 (1,024 captions) | Unlimited | 2GB | Markdown/HTML support |

*Varies by instance

---

### get_post

Get details of a specific post group.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID (e.g., `pg_abc123`) |

**Example prompts:**

```text
"Show me the details of post pg_abc123"
"What's the status of my last scheduled post?"
"Get info about my recent post"
```

**Python example:**

```python
async def get_post_details():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("get_post", {
                "postGroupId": "pg_abc123"
            })
            print(result.content[0].text)
```

---

### update_post

Reschedule or change post status.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID |
| `status` | string | No | New status: `draft` or `scheduled` |
| `scheduledTime` | string | No | New scheduled time (ISO 8601) |

**Example prompts:**

```text
"Reschedule post pg_abc123 to Friday at 3pm"
"Change my draft to scheduled"
"Move tomorrow's post to next week"
"Update the post to publish at 10am instead"
```

**Python example:**

```python
async def reschedule_post():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("update_post", {
                "postGroupId": "pg_abc123",
                "scheduledTime": "2026-03-01T15:00:00Z"
            })
            print(result.content[0].text)
```

---

### delete_post

Delete a post from all platforms.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID to delete |

**Example prompts:**

```text
"Delete post pg_abc123"
"Cancel my scheduled post for tomorrow"
"Remove all my draft posts"
```

**Python example:**

```python
async def delete_post():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("delete_post", {
                "postGroupId": "pg_abc123"
            })
            print(result.content[0].text)
```

---

### get_upload_url

Get a presigned URL to upload media files.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID to attach media to |
| `fileName` | string | Yes | File name (e.g., `photo.jpg`) |
| `contentType` | string | Yes | MIME type (e.g., `image/jpeg`, `video/mp4`) |
| `type` | string | Yes | Media type: `image` or `video` |

**Supported formats:**

| Type | Formats |
|------|---------|
| Images | JPEG, PNG, GIF, WebP |
| Videos | MP4, MOV, AVI, WebM |

**Python example:**

```python
import aiohttp

async def upload_image_to_post():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Get upload URL
            result = await session.call_tool("get_upload_url", {
                "postGroupId": "pg_abc123",
                "fileName": "product-photo.jpg",
                "contentType": "image/jpeg",
                "type": "image"
            })

            # Parse the upload URL from response
            upload_url = result.content[0].text  # Contains presigned S3 URL

            # Upload file using the presigned URL
            async with aiohttp.ClientSession() as http:
                with open("product-photo.jpg", "rb") as f:
                    await http.put(upload_url, data=f.read())
```

---

## Connections Tool

### list_connections

List all connected social media accounts.

**Parameters:** None

**Example prompts:**

```text
"Show my connected accounts"
"What platforms am I connected to?"
"List my social media accounts"
"Which accounts do I have linked?"
```

**Python example:**

```python
async def list_connections():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("list_connections", {})
            print(result.content[0].text)

asyncio.run(list_connections())
```

**Response example:**

```json
{
  "connections": [
    {
      "id": "twitter-123456",
      "platform": "twitter",
      "username": "@yourcompany",
      "displayName": "Your Company"
    },
    {
      "id": "linkedin-abc123",
      "platform": "linkedin",
      "username": "Your Company Page",
      "displayName": "Your Company"
    }
  ]
}
```

---

## LinkedIn Analytics Tools

### linkedin_post_stats

Get engagement metrics for a specific LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123456`) |
| `platformId` | string | Yes | Platform connection ID |
| `queryTypes` | string[] | No | Metrics to fetch: `IMPRESSION`, `MEMBERS_REACHED`, `RESHARE`, `REACTION`, `COMMENT` |

**Example prompts:**

```text
"How did my last LinkedIn post perform?"
"Get impressions for my LinkedIn post"
"Show engagement on my recent LinkedIn update"
"What's the reach of my LinkedIn announcement?"
```

**Python example:**

```python
async def get_linkedin_post_stats():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_post_stats", {
                "postedId": "urn:li:share:7123456789",
                "platformId": "linkedin-abc123",
                "queryTypes": ["IMPRESSION", "MEMBERS_REACHED", "RESHARE", "REACTION", "COMMENT"]
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "impressions": 1250,
  "uniqueImpressions": 680,
  "reactions": 28,
  "comments": 5,
  "shares": 3,
  "engagementRate": 2.9
}
```

---

### linkedin_account_stats

Get aggregated statistics for your LinkedIn account.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | Platform connection ID |
| `queryTypes` | string[] | No | Metrics to fetch |
| `aggregation` | string | No | `DAILY` or `TOTAL` (default: TOTAL) |

**Example prompts:**

```text
"Show my LinkedIn analytics"
"What's my total engagement on LinkedIn?"
"Give me daily LinkedIn stats for this week"
```

**Python example:**

```python
async def get_linkedin_account_stats():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_account_stats", {
                "platformId": "linkedin-abc123",
                "aggregation": "TOTAL"
            })
            print(result.content[0].text)
```

---

### linkedin_followers

Get follower count or growth over time.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | Platform connection ID |
| `period` | string | No | `lifetime` or `daily` |
| `dateRange` | object | No | For daily period. Structure: `{start: {year, month, day}, end: {year, month, day}}` |

**Example prompts:**

```text
"How many LinkedIn followers do I have?"
"Show my follower growth this month"
"What's my LinkedIn follower count?"
"Track my follower growth over the last 30 days"
```

**Python example:**

```python
async def get_linkedin_followers():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Lifetime followers
            result = await session.call_tool("linkedin_followers", {
                "platformId": "linkedin-abc123",
                "period": "lifetime"
            })
            print(result.content[0].text)

            # Daily growth
            result = await session.call_tool("linkedin_followers", {
                "platformId": "linkedin-abc123",
                "period": "daily",
                "dateRange": {
                    "start": {"year": 2026, "month": 2, "day": 1},
                    "end": {"year": 2026, "month": 2, "day": 28}
                }
            })
            print(result.content[0].text)
```

---

### linkedin_profile_summary

Get a combined profile overview with followers and stats.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | Platform connection ID |
| `dateRange` | object | No | Date range for stats. Structure: `{start: {year, month, day}, end: {year, month, day}}` |

**Example prompts:**

```text
"Give me a summary of my LinkedIn profile"
"How's my LinkedIn doing overall?"
"LinkedIn profile overview"
```

**Python example:**

```python
async def get_linkedin_summary():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_profile_summary", {
                "platformId": "linkedin-abc123"
            })
            print(result.content[0].text)
```

---

### linkedin_create_reaction

React to a LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN |
| `platformId` | string | Yes | Platform connection ID |
| `reactionType` | string | Yes | Reaction type (see below) |

**Reaction types:**

| Type | Description |
|------|-------------|
| `LIKE` | Standard like |
| `PRAISE` | Clapping hands |
| `EMPATHY` | Heart/love |
| `INTEREST` | Lightbulb/insightful |
| `APPRECIATION` | Thank you |
| `ENTERTAINMENT` | Funny/laughing |

**Example prompts:**

```text
"Like this LinkedIn post"
"React with PRAISE to post xyz"
"Add a heart reaction to my colleague's post"
```

**Python example:**

```python
async def react_to_post():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_create_reaction", {
                "postedId": "urn:li:share:7123456789",
                "platformId": "linkedin-abc123",
                "reactionType": "LIKE"
            })
            print(result.content[0].text)
```

---

### linkedin_delete_reaction

Remove a reaction from a LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN |
| `platformId` | string | Yes | Platform connection ID |

**Example prompts:**

```text
"Remove my reaction from this post"
"Unlike the LinkedIn post"
```

---

## Workspace Tools (B2B)

### list_workspace_users

List team members in your workspace.

**Parameters:** None

**Example prompts:**

```text
"Show my team members"
"Who's in my workspace?"
"List workspace users"
```

**Python example:**

```python
async def list_workspace_users():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("list_workspace_users", {})
            print(result.content[0].text)
```

---

### create_workspace_user

Add a new user to your workspace.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `username` | string | Yes | Email address |
| `displayName` | string | No | Display name for the user |

**Example prompts:**

```text
"Add john@example.com to my workspace"
"Create a new team member"
"Invite marketing@company.com to join"
```

**Python example:**

```python
async def add_workspace_user():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("create_workspace_user", {
                "username": "newuser@company.com",
                "displayName": "New User"
            })
            print(result.content[0].text)
```

---

### workspace_detach_user

Remove a user from your workspace.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `userId` | string | Yes | User ID to remove |

**Example prompts:**

```text
"Remove user xyz from my workspace"
"Detach this team member"
```

---

## Error Handling

All tools return errors in a consistent format:

```json
{
  "error": {
    "code": "INVALID_PARAMETER",
    "message": "Platform ID not found"
  }
}
```

**Common error codes:**

| Code | Description |
|------|-------------|
| `UNAUTHORIZED` | Invalid or missing API key |
| `INVALID_PARAMETER` | Invalid parameter value |
| `NOT_FOUND` | Resource not found |
| `RATE_LIMITED` | Too many requests |
| `PLATFORM_ERROR` | Error from social platform |

---

## Best Practices

1. **Always get connections first** — Use `list_connections` to get valid platform IDs before creating posts

2. **Use ISO 8601 dates** — All dates should be in format `2026-03-01T14:00:00Z`

3. **Respect character limits** — Check platform limits before creating posts

4. **Handle errors gracefully** — Check for error responses in tool results

5. **Batch operations** — When posting to multiple platforms, include all platform IDs in a single `create_post` call
