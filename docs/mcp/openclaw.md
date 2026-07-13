# OpenClaw AI Agent Integration

Connect [OpenClaw](https://docs.openclaw.ai) (open-source autonomous AI agent) to Publora for multi-platform social media posting.

## Prerequisites

- OpenClaw installed ([docs.openclaw.ai](https://docs.openclaw.ai))
- Publora account with API key (starts with `sk_`)
- At least one social media account connected in Publora

## Recommended: REST API

For autonomous agents like OpenClaw, we recommend using the **REST API directly** instead of MCP:

- ✅ Simple HTTP requests — no session management
- ✅ Standard headers — no special Accept header requirements
- ✅ Easier to debug — standard curl/httpie works
- ✅ More reliable — fewer moving parts

See [REST API examples](#rest-api-alternative) below for complete code.

## Alternative: MCP Protocol

MCP is better suited for interactive AI assistants (Claude Code, Cursor) where a human talks to the AI. If you still want to use MCP with OpenClaw:

### Configuration File (Recommended)

Create `config/mcporter.json`:

```json
{
  "mcpServers": {
    "publora": {
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

### mcporter CLI

```bash
mcporter list --config config/mcporter.json
```

> **Note:** Using CLI flags like `--header` may not work with all mcporter versions. Prefer the config file method.

> **Troubleshooting:**
> - If `mcporter list` reports *auth required* even with a Bearer header in this config, mcporter likely merged a different config source (`~/.claude.json`, `~/.mcporter/…`). Re-run with `--verbose` to see which file supplied the `publora` entry.
> - Don't run `mcporter auth publora`. `mcp.publora.com` *does* support OAuth 2.1 (DCR + PKCE), but its consent step is an **interactive** "paste your API key" web page — a headless CLI can't complete it. For OpenClaw/mcporter, authenticate with a static key header (`Authorization: Bearer sk_…` or `x-publora-key`) instead. (The OAuth flow is meant for the claude.ai web connector.)

## Using with OpenClaw

Once connected, talk to OpenClaw naturally:

```text
"Show my connected social accounts"
"Schedule a LinkedIn post for tomorrow at 9am"
"Post this announcement to all my platforms"
"How did my last post perform?"
```

OpenClaw will use Publora's MCP tools automatically.

## Autonomous Agent Example

Complete Python implementation for autonomous social media management:

```python
import asyncio
from datetime import datetime, timedelta
from contextlib import asynccontextmanager
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

@asynccontextmanager
async def publora_session(api_key: str):
    """Context manager for Publora MCP connection."""
    headers = {"Authorization": f"Bearer {api_key}"}
    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()
            yield session


async def get_connections(session):
    """Get all connected platforms."""
    result = await session.call_tool("list_connections", {})
    return result.content[0].text


async def schedule_post(session, content: str, platforms: list, scheduled_time: str):
    """Schedule a post to specified platforms."""
    result = await session.call_tool("create_post", {
        "content": content,
        "platforms": platforms,
        "scheduledTime": scheduled_time
    })
    return result.content[0].text


async def get_scheduled_posts(session):
    """List all scheduled posts."""
    result = await session.call_tool("list_posts", {
        "status": "scheduled"
    })
    return result.content[0].text


async def main():
    async with publora_session("sk_YOUR_API_KEY") as session:
        # Get connected platforms
        connections = await get_connections(session)
        print("Connected platforms:", connections)

        # Schedule a post for next Monday at 2pm UTC
        next_monday = datetime.now() + timedelta(days=(7 - datetime.now().weekday()) % 7)
        scheduled_time = next_monday.replace(
            hour=14, minute=0, second=0, microsecond=0
        ).isoformat() + "Z"

        result = await schedule_post(
            session,
            content="Automated post from OpenClaw agent!",
            platforms=["linkedin-YOUR_PLATFORM_ID"],
            scheduled_time=scheduled_time
        )
        print("Scheduled:", result)

        # Check scheduled posts
        posts = await get_scheduled_posts(session)
        print("Upcoming posts:", posts)


asyncio.run(main())
```

## REST API Alternative

For direct API access without MCP:

### Create Post (Node.js/TypeScript)

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'x-publora-key': 'sk_YOUR_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    content: 'Your post content here',
    platforms: ['linkedin-connection-id'],
    scheduledTime: new Date(Date.now() + 60000).toISOString(), // 1 minute from now
  }),
});

const data = await response.json();
console.log('Post created:', data);
```

### Create Post (Python)

```python
import requests
from datetime import datetime, timedelta

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={'x-publora-key': 'sk_YOUR_API_KEY'},
    json={
        'content': 'Post content',
        'platforms': ['linkedin-connection-id'],
        'scheduledTime': (datetime.utcnow() + timedelta(minutes=1)).isoformat() + 'Z',
    }
)

print('Post created:', response.json())
```

### Scheduled Post (Python)

```python
from datetime import datetime, timedelta
import requests

# Schedule for next Monday at 2pm UTC
next_monday = datetime.utcnow() + timedelta(days=(7 - datetime.utcnow().weekday()) % 7)
scheduled_time = next_monday.replace(hour=14, minute=0, second=0).isoformat() + 'Z'

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={'x-publora-key': 'sk_YOUR_API_KEY'},
    json={
        'content': 'Scheduled post content',
        'platforms': ['linkedin-connection-id'],
        'scheduledTime': scheduled_time,
    }
)

print('Post scheduled:', response.json())
```

### Media Upload

> **Fastest (one call):** pass `mediaUrls` (public https URLs) to `create-post` together with `scheduledTime` — Publora downloads and attaches the media server-side, then schedules. No upload round-trip.
>
> **Local files (draft → upload → schedule):** attaching media via `get-upload-url` to an *already-scheduled* post **demotes it back to `draft`** (`postGroupDemoted: true`). So create a **draft** (no `scheduledTime`), attach, then schedule with `update-post`. The step-by-step example below uses this flow.

```python
import requests
from datetime import datetime, timedelta

# Step 1: Create a DRAFT (omit scheduledTime) so media can attach without a demote
post_response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={'x-publora-key': 'sk_YOUR_API_KEY'},
    json={
        'content': 'Check out this image!',
        'platforms': ['linkedin-connection-id'],
        # No scheduledTime = draft
    }
)
post_data = post_response.json()
post_group_id = post_data['postGroupId']

# Step 2: Request presigned upload URL
upload_url_response = requests.post(
    'https://api.publora.com/api/v1/get-upload-url',
    headers={'x-publora-key': 'sk_YOUR_API_KEY'},
    json={
        'postGroupId': post_group_id,
        'fileName': 'image.jpg',
        'contentType': 'image/jpeg',
        'type': 'image'
    }
)
upload_data = upload_url_response.json()

# Step 3: Upload file to presigned URL
with open('image.jpg', 'rb') as f:
    requests.put(
        upload_data['uploadUrl'],
        data=f.read(),
        headers={'Content-Type': 'image/jpeg'}
    )

# Step 4 (optional): finalize/validate the upload early
requests.post(
    f'https://api.publora.com/api/v1/complete-media/{upload_data["mediaId"]}',
    headers={'x-publora-key': 'sk_YOUR_API_KEY'},
)

# Step 5: Schedule the now-media-bearing draft
requests.put(
    f'https://api.publora.com/api/v1/update-post/{post_group_id}',
    headers={'x-publora-key': 'sk_YOUR_API_KEY'},
    json={'status': 'scheduled', 'scheduledTime': (datetime.utcnow() + timedelta(hours=1)).isoformat() + 'Z'},
)
print('Image attached and post scheduled:', post_group_id)

# --- One-shot alternative: skip steps 2-5 entirely ---
# requests.post('https://api.publora.com/api/v1/create-post',
#     headers={'x-publora-key': 'sk_YOUR_API_KEY'},
#     json={'content': 'Check out this image!', 'platforms': ['linkedin-connection-id'],
#           'scheduledTime': (datetime.utcnow() + timedelta(hours=1)).isoformat() + 'Z',
#           'mediaUrls': ['https://example.com/photo.jpg']})
```

## Platform Capabilities

| Platform | Characters | Images | Video | Special Features |
|----------|------------|--------|-------|------------------|
| LinkedIn | 3,000 | 20 | 30 min / 500 MB | Documents (≤100 MB), multi-image, @mentions |
| X/Twitter | 280 (25K premium) | 4 | 2 min / 512 MB | Auto-threading |
| Instagram | 2,200 | 10 | Reels 3 min / 300 MB, 60s carousel | Reels & Stories, JPEG/PNG/WebP via API |
| Threads | 500 (10K with text attachment) | 20 | 5 min / 1 GB | Auto-threading |
| TikTok | 2,200 | N/A | up to 10 min (creator-dependent) | Video-only platform |
| YouTube | 100 title / 5,000 desc | N/A | 12 h | Shorts support |
| Facebook | 63,206 | 10 | 45 min / 2 GB | Page posts, Reels 90s / 1 GB |
| Bluesky | 300 | 4 | 3 min / 100 MB | Auto-facet detection |
| Mastodon | 500* | 4 | ~99 MB | Instance-variable |
| Telegram | 4,096 (1,024 captions) | 10 | 50 MB (Bot API) | Markdown/HTML |
| Pinterest | 100 title / 800 desc | 5 (carousel) | 15 min / 1 GB | Boards & idea pins *(in development)* |

*Varies by instance

## Production Best Practices

### Rate Limiting

```python
import time
from functools import wraps

def rate_limit(calls_per_minute=60):
    min_interval = 60.0 / calls_per_minute
    last_called = [0.0]

    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            elapsed = time.time() - last_called[0]
            if elapsed < min_interval:
                await asyncio.sleep(min_interval - elapsed)
            last_called[0] = time.time()
            return await func(*args, **kwargs)
        return wrapper
    return decorator

@rate_limit(calls_per_minute=30)
async def safe_api_call(session, tool_name, params):
    return await session.call_tool(tool_name, params)
```

### Retry with Exponential Backoff

```python
import asyncio
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return await func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    delay = base_delay * (2 ** attempt)
                    print(f"Attempt {attempt + 1} failed, retrying in {delay}s...")
                    await asyncio.sleep(delay)
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3)
async def reliable_post(agent, content, platforms, time):
    return await agent.schedule_post(content, platforms, time)
```

### Error Handling

```python
async def safe_schedule_post(agent, content, platforms, scheduled_time):
    try:
        result = await agent.schedule_post(content, platforms, scheduled_time)
        return {"success": True, "data": result}
    except Exception as e:
        error_msg = str(e)
        if "rate limit" in error_msg.lower():
            return {"success": False, "error": "Rate limited, try again later"}
        elif "unauthorized" in error_msg.lower():
            return {"success": False, "error": "Invalid API key"}
        elif "platform not found" in error_msg.lower():
            return {"success": False, "error": "Invalid platform ID"}
        else:
            return {"success": False, "error": error_msg}
```

## Next Steps

- [Tools Reference](./tools-reference.md) — All 13 MCP tools
- [Client Setup](./client-setup.md) — Other MCP clients
- [Examples](./examples.md) — More conversation examples
