# MCP Tools Reference

Complete reference for the 14 active Publora MCP tools with parameters, examples, and code snippets. Media can be attached two ways: the fast path (pass public **https** URLs via `mediaUrls` on `create_post`/`update_post`) or the upload dance (`get_upload_url` → HTTP PUT → `complete_media`). Three additional LinkedIn feed-retrieval tools (`linkedin_posts`, `linkedin_post_comments`, `linkedin_post_reactions`) are pending LinkedIn approval of the `r_member_social` permission — see [LinkedIn Feed Retrieval Tools](#linkedin-feed-retrieval-tools-coming-soon-requires-linkedin-approval) below. LinkedIn analytics and workspace-management features are available via the [REST OpenAPI reference](https://docs.publora.com/openapi.yaml), not MCP.

> **Note:** Most tools return the backend API object. `list_connections` deliberately wraps the backend list as `{ "connections": [...] }` for MCP structured content. `list_posts` also supports a `concise` mode that truncates content previews and adds response-format metadata.

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
| `responseFormat` | string | No | `detailed` (default) returns full post content; `concise` truncates each post's content to a preview to save tokens |

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
  "success": true,
  "posts": [
    {
      "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
      "content": "Excited to share our latest update!",
      "status": "scheduled",
      "scheduledTime": "2026-02-20T14:00:00Z",
      "platforms": [
        {
          "platformId": "linkedin-123456",
          "platform": "linkedin",
          "status": "scheduled"
        }
      ],
      "createdAt": "2026-02-19T10:30:00Z",
      "updatedAt": "2026-02-19T10:30:00Z",
      "mediaUrls": []
    }
  ],
  "pagination": {
    "totalItems": 1,
    "totalPages": 1,
    "page": 1,
    "limit": 20,
    "hasNextPage": false,
    "hasPrevPage": false
  }
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
| `scheduledTime` | string | No | When to publish (ISO 8601): `2026-03-01T14:00:00Z`. Omit to create a **draft**. |
| `mediaUrls` | string[] | No | Up to 10 public **https** image/video URLs, downloaded server-side and attached before validation. Pass together with `scheduledTime` to attach **and** schedule in one call. |
| `platformSettings` | object | No | Per-platform publishing options (see schema below). Strict — unknown platforms or keys are rejected. |
| `idempotencyKey` | string | No | Retry key (min length 1), forwarded as the `Idempotency-Key` header. Reusing it with the identical request replays the original response without creating another post. |

> **Draft behavior:** `scheduledTime` is optional in both MCP and REST — omit it to create a **draft**. Publishable media-required platforms (Instagram, TikTok, YouTube) must be created as a draft first (or given media via `mediaUrls`), then scheduled once media is attached. Pinterest is connect-only: passing media may satisfy registry validation, but it cannot be published because scheduler dispatch is not implemented.

> **Use `idempotencyKey` whenever a retry is possible.** An agent that retries after a network timeout has no way to know whether the first `create_post` landed — without a key it creates a **second post**. Pass a fresh unique key (e.g. a UUID) per distinct post; on retry, resend the *same* key with the *same* arguments and Publora replays the original result instead of posting again. Reusing a key with different arguments returns `422` `IDEMPOTENCY_KEY_CONFLICT`; retrying while the first call is still running returns `409` `IDEMPOTENCY_IN_FLIGHT` (wait and retry the identical call — do not switch keys).

> **A `scheduledTime` in the past is not taken literally.** Under 5 minutes late it is always clamped with `SCHEDULED_TIME_COERCED`. At 5+ minutes it is scheduled to become `400 SCHEDULED_TIME_IN_PAST` on 2026-08-25, unless production configuration overrides the date either way.

> **Publishable media-required platforms (Instagram, TikTok, YouTube):** scheduling one of these with no media fails validation with `MEDIA_REQUIRED` (HTTP 400, `{ "error": "Validation failed", "validation": {…} }`; the error's `suggestions` name the exact recovery tool calls). To satisfy it: pass `mediaUrls` in the same `create_post` call, **or** create a draft (omit `scheduledTime`), attach with `get_upload_url` → `complete_media`, then `update_post` with `status: "scheduled"`. Do not schedule Pinterest; it is connect-only.

> **`platformSettings` via MCP** — supported on `create_post` and `update_post`. The schema is **strict**: a mistyped platform or key (e.g. `coverUrl` → `coverurl`) is rejected with a validation error rather than silently dropped. These six platforms accept settings:
>
> ```json
> {
>   "platformSettings": {
>     "instagram": {
>       "videoType": "REELS | STORIES",
>       "shareToFeed": true,
>       "coverUrl": "https://.../cover.jpg"
>     },
>     "tiktok": {
>       "viewerSetting": "PUBLIC_TO_EVERYONE | MUTUAL_FOLLOW_FRIENDS | FOLLOWER_OF_CREATOR | SELF_ONLY",
>       "allowComments": true, "allowDuet": true, "allowStitch": true,
>       "commercialContent": false, "brandOrganic": false, "brandedContent": false
>     },
>     "youtube": {
>       "privacy": "public | unlisted | private",
>       "title": "string", "madeForKids": false,
>       "tags": ["string"], "categoryId": "string",
>       "playlist": { "id": "string", "platformId": "string" }
>     },
>     "threads": { "replyControl": "everyone | accounts_you_follow | mentioned_only" },
>     "telegram": { "disableNotification": false, "disableWebPagePreview": false, "protectContent": false },
>     "linkedin": {
>       "repostEnabled": true,
>       "repostParentUrn": "urn:li:share:123456",
>       "repostVisibility": "PUBLIC | CONNECTIONS"
>     }
>   }
> }
> ```
> For LinkedIn repost settings, `CONNECTIONS` is personal-profile-only. A company-page repost must use `PUBLIC` or scheduling returns `400`.
> YouTube custom thumbnails are **not** settable here (they need the separate multipart thumbnail endpoint, which MCP does not expose).

**Example prompts:**

```text
"Schedule 'Hello world!' to LinkedIn for tomorrow at 9am"
"Post 'We're hiring!' to Twitter and LinkedIn right now"
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

**Response example:**

```json
{
  "success": true,
  "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
  "scheduledTime": "2026-07-20T09:00:00.000Z"
}
```

**LinkedIn Mentions:**

You can @mention people and companies in LinkedIn posts using this syntax in your content:

```text
@{urn:li:person:ACoAABcD1234EfG|Serge Bulaev}           # Mention a person
@{urn:li:organization:107107343|Acme Corp Inc}  # Mention a company
```

**Example:**
```text
"Great insights from @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} at @{urn:li:organization:107107343|Creative Content Crafts Inc}!"
```

**Important:** The display name must exactly match the LinkedIn profile name (case-sensitive), including company suffixes like "Inc", "LLC", etc.

**Platform limits:**

<!-- synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->
| Platform | Characters | Images | Video | Special Features |
|----------|------------|--------|-------|------------------|
| LinkedIn | 3,000 | 10 | 500MB | Documents, @mentions |
| X/Twitter | 280 (25K premium) | 4 | 140s | Auto-threading |
| Instagram | 2,200 | 10 | 900s Reels, 3600s feed, 60s carousel | Reels & Stories supported |
| Threads | 500 (10,000 with text attachment) | 20 | 5min / 1 GB | Threading disabled |
| TikTok | 2,200 | 35 | 10min / 4 GB | Image carousel or video |
| YouTube | 5,000 desc | 0 | 12h / 256 GB | Shorts support |
| Facebook | 63,206 | 10 | 45min / 2 GB | Page posts, Reels |
| Bluesky | 300 | 4 | 3 min / 100 MB | Auto-facet detection |
| Mastodon | 500* | 4 | ~99 MB | Instance-variable |
| Telegram | 4,096 (1,024 captions) | 10 | 24h / 50 MB | Automatic Markdown parsing |

*Varies by instance

> **Note:** LinkedIn's "10 images" is the multi-image upload limit, not a carousel. Organic carousels on LinkedIn are **not** supported via the API (carousels are only available for sponsored/ad content). To share multi-page content organically, use LinkedIn document (PDF) posts instead.

> **Note:** Multi-threaded nested posts on Threads are temporarily unavailable. Single posts and carousel posts to Threads continue to work normally.

---

### get_post

Get details of a specific post group.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID (e.g., `67a1b2c3d4e5f6a7b8c9d0e1`) |

**Example prompts:**

```text
"Show me the details of post 67a1b2c3d4e5f6a7b8c9d0e1"
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
                "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1"
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true,
  "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
  "status": "scheduled",
  "scheduledTime": "2026-07-20T14:30:00.000Z",
  "platformSettings": {},
  "platforms": ["linkedin-abc123"],
  "posts": [
    {
      "_id": "67a1b2c3d4e5f6a7b8c9d0e2",
      "platform": "linkedin",
      "platformId": "abc123",
      "content": "Excited to share our latest update!",
      "status": "scheduled",
      "postedId": null,
      "permalink": null
    }
  ],
  "media": []
}
```

> **Note:** `get_post` returns group-level `status`, `scheduledTime`, `platformSettings`, `platforms`, and `media[]`, plus one `posts[]` entry per platform target. Each post includes nullable `platformId`, `postedId`, and `permalink`; internal thread-part IDs are not exposed.

---

### update_post

Reschedule or change post status.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID |
| `status` | string | No | New status: `draft` or `scheduled` |
| `scheduledTime` | string | No | New scheduled time (ISO 8601) |
| `mediaUrls` | string[] | No | Public https URLs (≤10) to download and **append** to the post's media. |
| `platformSettings` | object | No | Per-platform options to merge (same strict schema as `create_post`). |
| `idempotencyKey` | string | No | Retry key (min length 1), forwarded as the `Idempotency-Key` header. Reusing it with the identical request prevents repeated media appends and replays the original response. |

> **Note:** Provide at least one of `status`, `scheduledTime`, `mediaUrls`, or `platformSettings`. `update_post` is **not idempotent by default** — repeating a call with `mediaUrls` appends the same media a second time. Pass `idempotencyKey` to make a retry safe: the repeated call replays the original response instead of appending again.

> **`scheduledTime` handling:** omitting it keeps the current time. Under 5 minutes late is always clamped with `SCHEDULED_TIME_COERCED`; 5+ minutes is scheduled to become strict on 2026-08-25 unless production configuration overrides the date either way.

> **Known discrepancy:** the `update_post` tool description returned by the live MCP server still says a past `scheduledTime` "is snapped to now". That is only true under the conditions above; the note in this section is authoritative.

**Example prompts:**

```text
"Reschedule post 67a1b2c3d4e5f6a7b8c9d0e1 to Friday at 3pm"
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
                "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
                "scheduledTime": "2026-03-01T15:00:00Z"
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true,
  "message": "Post updated successfully",
  "scheduledTime": "2026-03-01T15:00:00.000Z",
  "postGroup": {
    "_id": "67a1b2c3d4e5f6a7b8c9d0e1",
    "status": "scheduled",
    "scheduledTime": "2026-03-01T15:00:00Z"
  }
}
```

> **Successful responses may carry `warnings`.** When present, `warnings` is an array of `{ code, message, ... }` objects (e.g. `SCHEDULED_TIME_COERCED`, which also carries `requested` and `effective`). The key is omitted entirely when there are no warnings. Surface these to the user — the call succeeded, but the API changed something about the request. Full list: [Error Handling](../guides/error-handling.md).

> **Note:** Top-level `scheduledTime` is always present and is `null` for drafts. The nested `postGroup.scheduledTime` is conditional and appears only when a scheduled time is set.

> **Note:** Only posts with status `draft` or `scheduled` can be updated. Attempting to update a post in any other status (e.g., `published`, `failed`, `partially_published`) returns a `400` error: `"Cannot update post: post is currently in {status} status"`.

---

### delete_post

Delete a post from all platforms.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | Post group ID to delete |

**Example prompts:**

```text
"Delete post 67a1b2c3d4e5f6a7b8c9d0e1"
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
                "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1"
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true
}
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

**Upload-layer acceptance:**

| Type | Formats |
|------|---------|
| Images | The MCP tool accepts an `image/*` MIME string; scheduling applies the target platform's format allowlist |
| Videos | The MCP tool accepts a `video/*` MIME string; scheduling applies the target platform's format allowlist |

> **⚠ Attaching media demotes a scheduled post to draft.** Calling `get_upload_url` on an already-`scheduled` post demotes it back to `draft` (`postGroupDemoted: true` in the response). Attach media on a **draft**, then schedule with `update_post`. Companion media tools: **`complete_media`** (finalize/validate an uploaded `mediaId` — optional, the scheduling gate probes lazily) and **`delete_media`** (remove one attached media file; also demotes a scheduled post). Media attached via `mediaUrls` needs neither.

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
                "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
                "fileName": "product-photo.jpg",
                "contentType": "image/jpeg",
                "type": "image"
            })

            # Parse the response - contains uploadUrl, fileUrl, and mediaId
            # Response: { "success": true, "uploadUrl": "https://...", "fileUrl": "https://...", "mediaId": "..." }
            import json
            data = json.loads(result.content[0].text)
            upload_url = data["uploadUrl"]

            # Upload file using the presigned URL
            async with aiohttp.ClientSession() as http:
                with open("product-photo.jpg", "rb") as f:
                    await http.put(upload_url, data=f.read())
```

### complete_media

Finalize a file uploaded via `get_upload_url` (probes the object, persists type/metadata). Call it after the presigned `PUT` succeeds. *(Optional — scheduling also finalizes pending media — but calling it early surfaces format/probe errors before publish.)* Not needed for media attached via `mediaUrls`.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mediaId` | string | Yes | The `mediaId` returned by `get_upload_url`. |

### delete_media

Remove a media slot from a post (detaches and deletes the underlying file). Deleting media from a **scheduled** post demotes it to `draft` — re-schedule with `update_post` afterward.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `mediaId` | string | Yes | The `mediaId` of the slot to remove (see the `media` array in `get_post`). |

> **Two ways to attach media:**
> 1. **Fast path** — pass `mediaUrls` (public https URLs) to `create_post`/`update_post`; the server downloads them. No upload steps.
> 2. **Upload dance** — `get_upload_url` → `PUT` the bytes to the presigned URL → `complete_media`. Use `delete_media` to drop a slot.

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
    "platformId": "twitter-123456789",
    "username": "@yourcompany",
    "displayName": "Your Company",
    "profileImageUrl": "https://pbs.twimg.com/profile_images/...",
    "profileUrl": null,
    "tokenStatus": "valid",
    "tokenExpiresIn": null,
    "accessTokenExpiresAt": null,
    "lastSuccessfulPost": null,
    "lastError": null,
    "subscriptionType": "Premium"
    },
    {
    "platformId": "linkedin-Tz9W5i6ZYG",
    "username": "Your Company Page",
    "displayName": "Your Company",
    "profileImageUrl": "https://media.licdn.com/...",
    "profileUrl": "https://www.linkedin.com/company/your-company",
    "tokenStatus": "valid",
    "tokenExpiresIn": "82d 4h",
    "accessTokenExpiresAt": "2026-06-15T12:00:00.000Z",
    "lastSuccessfulPost": "2026-03-10T09:30:00.000Z",
    "lastError": null,
    "subscriptionType": null
    }
  ]
}
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `platformId` | string | Unique ID for creating posts (e.g., `twitter-123456789`) |
| `username` | string | Platform username or handle |
| `displayName` | string | Display name on the platform |
| `profileImageUrl` | string | Profile image URL |
| `profileUrl` | string/null | URL to the profile on the platform |
| `tokenStatus` | string | Token health: `valid`, `expiring_soon`, `expired`, `unknown` |
| `tokenExpiresIn` | string/null | Human-readable time until expiration (e.g., "7d 3h") |
| `accessTokenExpiresAt` | string/null | ISO 8601 timestamp when the access token expires |
| `lastSuccessfulPost` | string/null | ISO 8601 timestamp of the last successful post via this connection |
| `lastError` | object/null | Last error details: `{ message: string, occurredAt: string }` |
| `subscriptionType` | string/null | Detected platform subscription tier when available (used for X Premium/PremiumPlus limits) |

---

## LinkedIn Reactions

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

## LinkedIn Comment Tools

### linkedin_create_comment

Post a comment on a LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123456` or `urn:li:ugcPost:123456`) |
| `platformId` | string | Yes | Platform connection ID |
| `message` | string | Yes | Raw input up to 10,000 characters; after mention processing, the text sent to LinkedIn must be at most 1,250 characters. Supports mentions: `@{urn:li:person:ID\|Name}` or `@{urn:li:organization:ID\|Company}` |
| `parentComment` | string | No | Parent comment URN for nested replies |

**Example prompts:**

```text
"Comment 'Great insights!' on this LinkedIn post"
"Post a comment on my latest LinkedIn update"
"Reply to this comment with 'Thanks for sharing!'"
"Comment on this post mentioning @{urn:li:person:ACoAABcD1234EfG|Jane Smith}"
```

**Python example:**

```python
async def comment_on_post():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_create_comment", {
                "postedId": "urn:li:ugcPost:7429953213384187904",
                "platformId": "linkedin-abc123",
                "message": "Great insights! Thanks for sharing."
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true,
  "comment": {
    "id": "7434695495614312448",
    "commentUrn": "urn:li:comment:(urn:li:ugcPost:xxx,7434695495614312448)",
    "message": "Great insights! Thanks for sharing."
  }
}
```

---

### linkedin_delete_comment

Delete a comment from a LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN the comment belongs to |
| `commentId` | string | Yes | Comment URN to delete |
| `platformId` | string | Yes | Platform connection ID |

**Example prompts:**

```text
"Delete my comment from this post"
"Remove the comment I just posted"
```

**Python example:**

```python
async def delete_comment():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_delete_comment", {
                "postedId": "urn:li:ugcPost:7429953213384187904",
                "commentId": "urn:li:comment:(urn:li:ugcPost:xxx,7434695495614312448)",
                "platformId": "linkedin-abc123"
            })
            print(result.content[0].text)
```

---

## LinkedIn Reshare Tool

### linkedin_create_reshare

Reshare an existing LinkedIn post to your feed, optionally with commentary.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | LinkedIn platform connection ID |
| `parent` | string | Yes | Post URN to reshare, such as `urn:li:share:123456` or `urn:li:ugcPost:123456` |
| `commentary` | string | No | Commentary shown above the reshare (maximum 3,000 characters) |
| `visibility` | string | No | `PUBLIC` or `CONNECTIONS` (default: `PUBLIC`; `CONNECTIONS` is for personal accounts) |

**Example prompt:**

```text
"Reshare urn:li:share:123456 on linkedin-abc123 with the commentary 'Worth reading'"
```

---

## LinkedIn Feed Retrieval Tools (Coming Soon — Requires LinkedIn Approval)

> **Status: DISABLED** - These tools are not yet available. They require the `r_member_social` permission, which is **RESTRICTED** and requires LinkedIn approval. The implementation is ready and will be enabled once LinkedIn approves the permission for Publora.

### linkedin_posts

Retrieve posts authored by a LinkedIn account.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | Platform connection ID (e.g., `linkedin-XxxYyy`) |
| `start` | number | No | Starting index for pagination (default: 0) |
| `count` | number | No | Number of posts to retrieve (default: 10, max: 100) |
| `sortBy` | string | No | Sort order: `LAST_MODIFIED` or `CREATED` (default: LAST_MODIFIED) |

**Example prompts:**

```text
"Show my recent LinkedIn posts"
"Get my last 10 LinkedIn posts"
"What have I posted on LinkedIn this month?"
"List my LinkedIn content"
```

**Python example:**

```python
async def get_my_linkedin_posts():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_posts", {
                "platformId": "linkedin-abc123",
                "count": 20,
                "sortBy": "LAST_MODIFIED"
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true,
  "posts": [
    {
      "id": "urn:li:share:7123456789",
      "commentary": "Excited to share our latest product update!",
      "publishedAt": 1709294400000,
      "visibility": "PUBLIC",
      "lifecycleState": "PUBLISHED"
    }
  ],
  "pagination": {
    "start": 0,
    "count": 10,
    "total": 45
  },
  "cached": false
}
```

---

### linkedin_post_comments

Retrieve comments on a specific LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123456` or `urn:li:ugcPost:123456`) |
| `platformId` | string | Yes | Platform connection ID |
| `start` | number | No | Starting index for pagination (default: 0) |
| `count` | number | No | Number of comments to retrieve (default: 20, max: 100) |

**Example prompts:**

```text
"Show comments on my last LinkedIn post"
"Get all comments on this LinkedIn post"
"What are people saying about my post?"
"List comments on post urn:li:share:123456"
```

**Python example:**

```python
async def get_post_comments():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_post_comments", {
                "postedId": "urn:li:ugcPost:7429953213384187904",
                "platformId": "linkedin-abc123",
                "count": 50
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true,
  "comments": [
    {
      "id": "6636062862760562688",
      "commentUrn": "urn:li:comment:(urn:li:activity:123,6636062862760562688)",
      "actor": "urn:li:person:xxx",
      "message": "Great insights! Thanks for sharing.",
      "created": { "time": 1582160678569 },
      "likesSummary": { "totalLikes": 5 },
      "commentsCount": 2
    }
  ],
  "pagination": {
    "start": 0,
    "count": 20,
    "total": 89
  },
  "cached": false
}
```

---

### linkedin_post_reactions

Retrieve reactions on a specific LinkedIn post.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123456` or `urn:li:ugcPost:123456`) |
| `platformId` | string | Yes | Platform connection ID |
| `start` | number | No | Starting index for pagination (default: 0) |
| `count` | number | No | Number of reactions to retrieve (default: 50, max: 100) |

**Example prompts:**

```text
"Who liked my LinkedIn post?"
"Show reactions on this post"
"Get all reactions on my announcement"
"List people who reacted to my post"
```

**Python example:**

```python
async def get_post_reactions():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            result = await session.call_tool("linkedin_post_reactions", {
                "postedId": "urn:li:ugcPost:7429953213384187904",
                "platformId": "linkedin-abc123",
                "count": 100
            })
            print(result.content[0].text)
```

**Response example:**

```json
{
  "success": true,
  "reactions": [
    {
      "id": "urn:li:reaction:(urn:li:person:xxx,urn:li:activity:123)",
      "reactionType": "LIKE",
      "actor": "urn:li:person:xxx",
      "created": { "time": 1686183251857 }
    },
    {
      "id": "urn:li:reaction:(urn:li:person:yyy,urn:li:activity:123)",
      "reactionType": "PRAISE",
      "actor": "urn:li:person:yyy",
      "created": { "time": 1686183300000 }
    }
  ],
  "pagination": {
    "start": 0,
    "count": 50,
    "total": 456
  },
  "cached": false
}
```

**Reaction types returned:**

| Type | Description |
|------|-------------|
| `LIKE` | Standard thumbs up |
| `PRAISE` | Clapping hands |
| `EMPATHY` | Heart/love |
| `INTEREST` | Lightbulb (insightful) |
| `APPRECIATION` | Thank you |
| `ENTERTAINMENT` | Funny/laughing |

---

## Error Handling

Every successful tool result carries both a text block and a machine-readable `structuredContent` object (MCP spec). On failure the tool surfaces the backend's full structured payload — `code`, `platform`, and `suggestions` where available — not just a bare message. Common errors include:

| Error Message | Cause |
|---------------|-------|
| `"API key required..."` | Missing or invalid API key |
| `"Platform ID not found"` | Invalid platform connection ID |
| `"Post not found"` | Invalid post group ID |
| `"API error: 429"` | Rate limited by the API |
| `"API error: 502"` | Platform API temporarily unavailable |

**Example error handling in Python:**

```python
try:
    result = await session.call_tool("create_post", {
        "content": "Hello world!",
        "platforms": ["linkedin-abc123"],
        "scheduledTime": "2026-03-01T14:00:00Z"
    })
    print(result.content[0].text)
except Exception as e:
    print(f"Tool error: {e}")
```

---

## Best Practices

1. **Always get connections first** — Use `list_connections` to get valid platform IDs before creating posts

2. **Use ISO 8601 dates** — All dates should be in format `2026-03-01T14:00:00Z`

3. **Respect character limits** — Check platform limits before creating posts

4. **Handle errors gracefully** — Check for error responses in tool results

5. **Batch operations** — When posting to multiple platforms, include all platform IDs in a single `create_post` call
