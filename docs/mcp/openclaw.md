# OpenClaw AI Agent Integration

Connect [OpenClaw](https://docs.openclaw.ai) (open-source autonomous AI agent) to Publora for multi-platform social media posting through natural conversation.

## Prerequisites

- OpenClaw installed ([docs.openclaw.ai](https://docs.openclaw.ai))
- Publora account with API key (starts with `sk_`)
- At least one social media account connected in Publora

## Setup Methods

### Method 1: mcporter CLI (Recommended)

mcporter is a CLI tool for connecting MCP servers to OpenClaw.

**List available tools:**

```bash
mcporter list --http-url https://mcp.publora.com --name publora
```

**Persist configuration:**

```bash
mcporter list --http-url https://mcp.publora.com --name publora --persist config/mcporter.json
```

**With authentication:**

```bash
mcporter list \
  --http-url https://mcp.publora.com \
  --name publora \
  --header "Authorization: Bearer sk_YOUR_API_KEY" \
  --persist config/mcporter.json
```

### Method 2: Configuration File

Create or edit `config/mcporter.json`:

```json
{
  "servers": {
    "publora": {
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

### Method 3: Environment Variable

```bash
export PUBLORA_API_KEY="sk_YOUR_API_KEY"
```

Then in config:

```json
{
  "servers": {
    "publora": {
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer ${PUBLORA_API_KEY}"
      }
    }
  }
}
```

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
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

class PubloraAgent:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.session = None

    async def connect(self):
        """Initialize MCP connection."""
        headers = {"Authorization": f"Bearer {self.api_key}"}
        self.transport = streamablehttp_client(
            "https://mcp.publora.com",
            headers=headers
        )
        read, write, _ = await self.transport.__aenter__()
        self.session = ClientSession(read, write)
        await self.session.__aenter__()
        await self.session.initialize()
        return self

    async def get_connections(self):
        """Get all connected platforms."""
        result = await self.session.call_tool("list_connections", {})
        return result.content[0].text

    async def schedule_post(self, content: str, platforms: list, scheduled_time: str):
        """Schedule a post to specified platforms."""
        result = await self.session.call_tool("create_post", {
            "content": content,
            "platforms": platforms,
            "scheduledTime": scheduled_time
        })
        return result.content[0].text

    async def get_scheduled_posts(self):
        """List all scheduled posts."""
        result = await self.session.call_tool("list_posts", {
            "status": "scheduled"
        })
        return result.content[0].text

    async def get_linkedin_analytics(self, platform_id: str):
        """Get LinkedIn profile analytics."""
        result = await self.session.call_tool("linkedin_profile_summary", {
            "platformId": platform_id
        })
        return result.content[0].text

    async def close(self):
        """Close the connection."""
        if self.session:
            await self.session.__aexit__(None, None, None)
        if self.transport:
            await self.transport.__aexit__(None, None, None)


async def main():
    agent = PubloraAgent("sk_YOUR_API_KEY")
    await agent.connect()

    try:
        # Get connected platforms
        connections = await agent.get_connections()
        print("Connected platforms:", connections)

        # Schedule a post for next Monday at 2pm UTC
        next_monday = datetime.now() + timedelta(days=(7 - datetime.now().weekday()) % 7)
        scheduled_time = next_monday.replace(
            hour=14, minute=0, second=0, microsecond=0
        ).isoformat() + "Z"

        result = await agent.schedule_post(
            content="Automated post from OpenClaw agent!",
            platforms=["linkedin-YOUR_PLATFORM_ID"],
            scheduled_time=scheduled_time
        )
        print("Scheduled:", result)

        # Check scheduled posts
        posts = await agent.get_scheduled_posts()
        print("Upcoming posts:", posts)

    finally:
        await agent.close()


asyncio.run(main())
```

## REST API Alternative

For direct API access without MCP:

### Create Post (Node.js/TypeScript)

```javascript
const response = await fetch('https://api.publora.com/v1/posts', {
  method: 'POST',
  headers: {
    'Authorization': 'Bearer sk_YOUR_API_KEY',
    'Content-Type': 'application/json',
  },
  body: JSON.stringify({
    content: 'Your post content here',
    platforms: ['linkedin-connection-id'],
    status: 'published',
  }),
});

const data = await response.json();
console.log('Post created:', data);
```

### Create Post (Python)

```python
import requests

response = requests.post(
    'https://api.publora.com/v1/posts',
    headers={'Authorization': 'Bearer sk_YOUR_API_KEY'},
    json={
        'content': 'Post content',
        'platforms': ['linkedin-connection-id'],
        'status': 'published',
    }
)

print('Post created:', response.json())
```

### Scheduled Post (Python)

```python
from datetime import datetime, timedelta
import requests

# Schedule for next Monday at 2pm UTC
next_monday = datetime.now() + timedelta(days=(7 - datetime.now().weekday()) % 7)
scheduled_time = next_monday.replace(hour=14, minute=0, second=0).isoformat() + 'Z'

response = requests.post(
    'https://api.publora.com/v1/posts',
    headers={'Authorization': 'Bearer sk_YOUR_API_KEY'},
    json={
        'content': 'Scheduled post content',
        'platforms': ['linkedin-connection-id'],
        'status': 'scheduled',
        'scheduledTime': scheduled_time,
    }
)

print('Post scheduled:', response.json())
```

### Media Upload (3-step process)

```python
import requests

# Step 1: Request presigned upload URL
upload_url_response = requests.post(
    'https://api.publora.com/v1/uploads',
    headers={'Authorization': 'Bearer sk_YOUR_API_KEY'},
    json={
        'fileName': 'image.jpg',
        'contentType': 'image/jpeg',
        'type': 'image'
    }
)
upload_data = upload_url_response.json()

# Step 2: Upload file to presigned URL
with open('image.jpg', 'rb') as f:
    requests.put(
        upload_data['uploadUrl'],
        data=f.read(),
        headers={'Content-Type': 'image/jpeg'}
    )

# Step 3: Use file URL in post
response = requests.post(
    'https://api.publora.com/v1/posts',
    headers={'Authorization': 'Bearer sk_YOUR_API_KEY'},
    json={
        'content': 'Check out this image!',
        'platforms': ['linkedin-connection-id'],
        'mediaUrls': [upload_data['fileUrl']],
        'status': 'published',
    }
)
```

## Platform Capabilities

| Platform | Characters | Images | Video | Special Features |
|----------|------------|--------|-------|------------------|
| LinkedIn | 3,000 | 20 | 200MB | Documents, carousels |
| X/Twitter | 280 (25K premium) | 4 | 140s | Auto-threading |
| Instagram | 2,200 | 10 | 90s | Reels supported |
| Threads | 500 | 10 | 5min | Auto-threading |
| TikTok | 2,200 | 35 carousel | 10min | Video-focused |
| YouTube | 5,000 desc | N/A | 12h | Shorts support |
| Facebook | 63,206 | 10 | 240min | Page posts, Reels |
| Bluesky | 300 | 4 | N/A | Auto-facet detection |
| Mastodon | 500* | 4 | 40MB | Instance-variable |
| Telegram | 4,096 | Unlimited | 2GB | Markdown/HTML |

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

- [Tools Reference](./tools-reference.md) — All 16 MCP tools
- [Client Setup](./client-setup.md) — Other MCP clients
- [Examples](./examples.md) — More conversation examples
