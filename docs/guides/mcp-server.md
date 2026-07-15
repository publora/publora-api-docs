# Publora MCP Server

Control Publora directly from AI assistants like Claude Code, Claude Desktop, Cursor, and any MCP-compatible client. No coding required -- just describe what you want in plain English.

## Overview

[Model Context Protocol (MCP)](https://modelcontextprotocol.io) is an open standard that lets AI assistants connect to external tools and services. Publora provides a remote MCP server at `mcp.publora.com` -- one-time setup, then just talk to your AI.

**Perfect for:**
- **Marketers** -- schedule campaigns, check analytics, manage multiple accounts
- **Content creators** -- post to all platforms at once without leaving your AI chat
- **Business owners** -- delegate social media tasks to your AI assistant
- **Developers** -- integrate Publora into AI-powered workflows

**Just describe what you want:**

> "Show my connected social accounts"
> "Schedule a post to LinkedIn for tomorrow at 9am"
> "Delete my scheduled post for tomorrow"
> "Post this announcement to all my accounts"

## Quick Start

### 1. Get Your API Key

Log in to [Publora](https://publora.com) → **API** in the sidebar → Copy your key (`sk_...`). Click **MCP** in the sidebar for additional setup instructions.

### 2. Configure Your MCP Client

Add Publora to your MCP configuration:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

> **Note:** Both `https://mcp.publora.com/` and `https://mcp.publora.com/mcp` work as the MCP endpoint URL.

### 3. Restart Your Client

After restarting, you'll have access to 14 Publora tools. Try asking:

> "List my connected social media accounts"

---

## Client-Specific Setup

### Claude Code (CLI)

Edit `~/.claude.json` or create `.mcp.json` in your project:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

Restart Claude Code. Verify with `/mcp` command.

### Claude Desktop

1. Open Claude Desktop → **Settings** → **Developer** → **Edit Config**
2. Add the Publora server configuration (same JSON as above)
3. Restart Claude Desktop

### Cursor

Create `.cursor/mcp.json` in your project:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

### Using with Other MCP Servers

MCP servers don't conflict -- Claude loads all servers and merges their tools. Example with Context7:

```json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"]
    },
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

---

## Available Tools (14)

### Posts

| Tool | Description |
|------|-------------|
| `list_posts` | List posts with filters (status, platform, dates, pagination; `responseFormat: concise` for token-lean previews) |
| `create_post` | Create a draft or schedule a post; accepts `mediaUrls` (fast media path) and `platformSettings` |
| `get_post` | Get post group details |
| `update_post` | Change status/schedule, append `mediaUrls`, or merge `platformSettings` |
| `delete_post` | Delete a post group and all platform posts |

### Media

| Tool | Description |
|------|-------------|
| `get_upload_url` | Get a presigned S3 URL for media upload |
| `complete_media` | Finalize a file uploaded via `get_upload_url` |
| `delete_media` | Remove a media slot from a post |

### Platform Connections

| Tool | Description |
|------|-------------|
| `list_connections` | List all connected social media accounts |

### LinkedIn Engagement

| Tool | Description |
|------|-------------|
| `linkedin_create_reaction` | React to a post (LIKE, PRAISE, EMPATHY, etc.) |
| `linkedin_delete_reaction` | Remove a reaction |
| `linkedin_create_comment` | Post a comment on a LinkedIn post |
| `linkedin_delete_comment` | Delete a comment from a LinkedIn post |
| `linkedin_create_reshare` | Reshare a LinkedIn post, optionally with commentary and visibility settings |

> LinkedIn **analytics/followers/profile-summary** and **workspace** management are **not** MCP tools — use the [REST OpenAPI reference](https://docs.publora.com/openapi.yaml). LinkedIn feed-retrieval tools (`linkedin_posts`, `linkedin_post_comments`, `linkedin_post_reactions`) are implemented but disabled pending LinkedIn's `r_member_social` approval.

---

## Example Conversations

> **Note:** Analytics examples below illustrate REST-API capabilities. MCP itself exposes engagement *actions* (reactions/comments), not analytics reads — pull metrics via the [REST OpenAPI reference](https://docs.publora.com/openapi.yaml).

### Schedule a Post

> **You:** Schedule a post to my LinkedIn saying "Excited to announce our new product launch!" for tomorrow at 9am EST

> **Claude:** I'll schedule that post for you. Let me first check your connected accounts...
>
> Done! Post scheduled to LinkedIn for 2026-02-20 at 9am EST.

### Cross-Platform Posting

> **You:** Post "We're hiring! Check out our careers page" to Twitter, LinkedIn, and Bluesky right now

> **Claude:** I'll post that to all three platforms...
>
> Done! Published to:
> - Twitter: https://twitter.com/...
> - LinkedIn: https://linkedin.com/...
> - Bluesky: https://bsky.app/...

### Check Post Analytics

> **You:** How did my last LinkedIn post perform?

> **Claude:** Your post from Feb 15 received:
> - 1,250 impressions
> - 680 unique viewers
> - 28 reactions, 5 comments, 3 reshares
> - Engagement rate: 2.9%

### Content Planning

> **You:** Show me all my scheduled posts for next week

> **Claude:** You have 5 scheduled posts:
> 1. Mon 9am - LinkedIn: "Product update..."
> 2. Tue 2pm - Twitter + LinkedIn: "Blog post..."
> 3. Wed 10am - LinkedIn: "Case study..."
> 4. Thu 3pm - Twitter: "Quick tip..."
> 5. Fri 9am - LinkedIn + Bluesky: "Week recap..."

### Campaign Management

> **You:** Cancel all my draft posts and show me what's still scheduled

> **Claude:** Deleted 3 draft posts. You still have 2 scheduled:
> - Tomorrow 10am - LinkedIn: "Q1 results..."
> - Friday 2pm - Twitter: "Weekend tips..."

### Weekly Review

> **You:** Give me a summary of how my LinkedIn performed this week

> **Claude:** This week's LinkedIn performance:
> - 5 posts published
> - 4,230 total impressions
> - 89 reactions, 12 comments
> - Best performer: "Product launch" (1,850 impressions)
> - Follower growth: +47

---

## Python Client Example

For programmatic access or testing:

```python
import asyncio
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # List tools
            tools = await session.list_tools()
            print(f"Available: {len(tools.tools)} tools")

            # Get connections
            result = await session.call_tool("list_connections", {})
            print(result.content[0].text)

asyncio.run(main())
```

Install dependencies:

```bash
pip install mcp httpx
```

---

## Verification

### Health Check

```bash
curl https://mcp.publora.com/health
# {"status":"ok","service":"publora-mcp"}
```

### Test MCP Handshake

```bash
curl -X POST https://mcp.publora.com \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer sk_YOUR_API_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0.0"}}}'
```

---

## Authentication

Every request requires your Publora API key via one of:

- `Authorization: Bearer sk_...` (recommended)
- `x-publora-key: sk_...`

This is the same key used for the REST API.

---

## HTTP Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| POST | `/` | MCP messages (Streamable HTTP) |
| GET | `/` | SSE stream (server notifications) |
| DELETE | `/` | Close session |
| GET | `/health` | Health check |

---

## Troubleshooting

### "API key required" Error

Ensure your Authorization header is correctly formatted:
- Correct: `Authorization: Bearer sk_abc123...`
- Wrong: `Authorization: sk_abc123...`

### Tools Not Showing

1. Check your MCP config syntax (valid JSON)
2. Verify the `type` is `"http"` (not `"url"`)
3. Restart your MCP client completely
4. Run `/mcp` in Claude Code to verify connection

### Session Errors

MCP uses sessions for stateful connections. If you see "Invalid or missing session", the server will automatically create a new one on the next request.

---

## Comparison: MCP vs REST API

| Feature | MCP Server | REST API |
|---------|------------|----------|
| **Interface** | Natural language via AI | HTTP requests |
| **Best for** | Interactive exploration, quick tasks | Programmatic integration, automation |
| **Setup** | One-time config | Per-request auth |
| **Tools** | 14 MCP tools | Full API access |
| **Rate limits** | Same as REST API | Same |

Use MCP for conversational workflows. Use REST API for production integrations.

---

*[Publora](https://publora.com) -- Social media API with free Starter plan, paid plans from Pro ($2.99/mo billed yearly)*
