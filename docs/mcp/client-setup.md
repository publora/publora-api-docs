# MCP Client Setup Guide

Detailed setup instructions for connecting Publora MCP to different AI clients.

## Prerequisites

1. **Publora account** — Sign up at [publora.com](https://publora.com)
2. **Connected social account** — At least one platform connected in the Publora dashboard
3. **API key** — only for clients that authenticate with a static key. Get it from **API** in the sidebar (starts with `sk_`). Claude and other clients that sign in with OAuth don't need one.

> **Both auth paths work.** OAuth is the default in Claude — the client opens a browser window and you never paste a key — and Claude Code, Codex/ChatGPT, Cursor, VS Code, Manus and other browser-capable clients can complete the same flow. Static API-key headers (`Authorization: Bearer sk_...` or `x-publora-key: sk_...`) remain fully supported for any client that can send a header, and are the only option for headless / non-interactive environments.

> **Auth headers:** Direct REST API calls to `api.publora.com` require `x-publora-key: sk_...`. MCP accepts `Authorization: Bearer sk_...` (recommended for MCP clients) and also `x-publora-key: sk_...`. The key-based examples below use the recommended Bearer format.

> **MCP client header:** The MCP server automatically sends an `x-publora-client: "mcp"` header on internal API calls. This identifies MCP traffic server-side. Your account must have `mcpAccess` enabled. All standard plans — including the free Starter plan — have MCP access; only custom accounts with `mcpAccess` disabled will receive a `403` error.

---

## Claude (web, desktop and mobile)

Publora is in the Claude connectors directory. Open the listing and select **Connect**:

**[claude.ai/directory/publora](https://claude.ai/directory/publora)**

1. Claude opens a Publora window. Sign in to Publora if you aren't already; the consent page reads *"An application is requesting access to your Publora account"* and shows which account you are authorizing as.
2. Click **Approve**. Publora mints a dedicated API key for this connector (named `MCP (Claude #<id>)`) and hands it to Claude as the access token — **you never paste a key**. Manage or revoke it any time on the **API** page in your dashboard.
3. The 18 Publora tools appear in the connector. Re-authorize the same way if you revoke the key.

No API key, no config file, no URL to paste. The connector belongs to your Claude account, so the same connection works in Claude on the web, in Claude Desktop and in the Claude mobile apps.

### Adding it by URL instead

You can still add Publora as a custom connector by hand:

1. In Claude, open **Customize → Connectors** ([claude.ai/customize/connectors](https://claude.ai/customize/connectors)), click **+** and choose **Add custom connector**.
2. Name it **Publora** and enter the URL `https://mcp.publora.com/mcp`.
3. Leave **OAuth Client ID** and **OAuth Client Secret** under **Advanced settings** empty. Claude registers itself with Publora automatically through **OAuth 2.1** (Dynamic Client Registration + PKCE).
4. Click **Add**, then **Connect** — the same Publora window opens.

---

## Claude Desktop

Claude Desktop uses the same connectors as Claude on the web: open [the listing](https://claude.ai/directory/publora) and select **Connect**, or add Publora by URL as described above. There is no config file to edit. If Claude Desktop was already running when you connected on the web, restart it to see Publora.

The **Settings → Developer → Edit Config** file (`claude_desktop_config.json`) is for local MCP servers that run on your own machine. Publora is a remote server, so you don't need it.

### Verification

Start a new conversation and ask:

```text
"What MCP tools do you have available?"
```

Claude should list the 18 active Publora tools, including `post_stats`, `profile_stats`, `linkedin_create_reshare` and `linkedin_list_mentionables` (3 additional LinkedIn feed-retrieval tools — `linkedin_posts`, `linkedin_post_comments`, `linkedin_post_reactions` — are pending LinkedIn approval).

---

## Claude Code (CLI)

Claude Code is Anthropic's command-line interface for Claude.

### Option 1: Sign in with OAuth (recommended)

```bash
claude mcp add --transport http --scope user publora https://mcp.publora.com/mcp
```

Then run `/mcp` inside Claude Code, select **publora**, choose **Authenticate** and sign in to Publora in the browser window that opens. No API key needed. `--scope user` makes Publora available in every project; leave it out to add Publora to the current project only.

### Option 2: CLI Command with an API Key

For CI, a shared machine, or any environment that cannot open a browser, send a static key instead:

```bash
claude mcp add --transport http --scope user publora https://mcp.publora.com/mcp \
  --header "Authorization: Bearer sk_YOUR_API_KEY"
```

### Option 3: Global Configuration

Edit `~/.claude.json`:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com/mcp",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

### Option 4: Project Configuration

Create `.mcp.json` in your project root:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com/mcp",
      "headers": {
        "Authorization": "Bearer sk_YOUR_API_KEY"
      }
    }
  }
}
```

Leave out the `headers` block in either file to sign in with OAuth through `/mcp` instead.

### Verification

After restarting Claude Code:

```bash
# Check connected MCP servers
/mcp

# Test Publora connection
"Show my connected social accounts"
```

---

## Cursor

Cursor is an AI-powered code editor with MCP support.

### Project Configuration

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

### Global Configuration

For all projects, edit `~/.cursor/mcp.json`:

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

### Verification

1. Restart Cursor
2. Open the AI chat
3. Ask: "List my Publora connections"

---

## VS Code / JetBrains / Other MCP Clients

Publora MCP is a standard MCP Streamable HTTP endpoint. Most MCP clients accept the same shape with minor key-name differences:

- **URL:** `https://mcp.publora.com`
- **Header:** `Authorization: Bearer sk_YOUR_API_KEY` (or `x-publora-key: sk_YOUR_API_KEY`)
- **Transport:** HTTP (Streamable HTTP)

Refer to your client's MCP configuration docs for the exact config-file path and top-level JSON key (commonly `mcpServers`, `servers`, or a client-specific path). The Claude Code, Cursor, and mcporter examples on this page cover the most common config shapes.

---

## Windsurf

Windsurf is an AI-powered IDE by Codeium.

### Configuration

Create or edit `.windsurf/mcp.json`:

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

---

## Manus

Publora is a **published connector in [Manus](https://manus.im)' own directory** — nothing to install, no config file. Add it from **Settings → Connectors → Browse Connectors → Apps**, then authorize with your API key (some builds offer Publora's OAuth sign-in instead).

See [Manus Integration](./manus.md) for the full walkthrough, including the direct connector link and the mobile caveat.

---

## OpenClaw / mcporter CLI

[mcporter](https://github.com/openclaw/mcporter) is a CLI tool for connecting MCP servers to OpenClaw and other agents.

### List Available Tools

```bash
mcporter list --http-url https://mcp.publora.com --name publora
```

### With Authentication

```bash
mcporter list \
  --http-url https://mcp.publora.com \
  --name publora \
  --header "Authorization: Bearer sk_YOUR_API_KEY"
```

### Persist Configuration

```bash
mcporter list \
  --http-url https://mcp.publora.com \
  --name publora \
  --header "Authorization: Bearer sk_YOUR_API_KEY" \
  --persist config/mcporter.json
```

### Configuration File

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

See [OpenClaw Integration Guide](./openclaw.md) for complete autonomous agent examples.

---

## Multiple MCP Servers

You can use Publora alongside other MCP servers. They don't conflict — tools are merged.

### Example: Publora + Context7

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

### Example: Publora + GitHub + Filesystem

```json
{
  "mcpServers": {
    "github": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_TOKEN": "ghp_YOUR_TOKEN"
      }
    },
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/path/to/files"]
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

## Programmatic Access (Python)

For testing or custom integrations, use Python:

### Installation

```bash
pip install mcp httpx
```

### Basic Example

```python
import asyncio
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # List available tools
            tools = await session.list_tools()
            print(f"Available tools: {len(tools.tools)}")
            for tool in tools.tools:
                print(f"  - {tool.name}: {tool.description}")

            # Get connections
            result = await session.call_tool("list_connections", {})
            print(result.content[0].text)

asyncio.run(main())
```

### Full Workflow Example

```python
import asyncio
from datetime import datetime, timedelta
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def schedule_post_workflow():
    """Complete workflow: check connections, create post, verify."""
    headers = {"Authorization": "Bearer sk_YOUR_API_KEY"}

    async with streamablehttp_client("https://mcp.publora.com", headers=headers) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()

            # Step 1: Get connections
            print("Getting connections...")
            connections = await session.call_tool("list_connections", {})
            print(connections.content[0].text)

            # Step 2: Schedule a post
            tomorrow = (datetime.utcnow() + timedelta(days=1)).replace(
                hour=9, minute=0, second=0, microsecond=0
            ).isoformat() + "Z"

            print(f"\nScheduling post for {tomorrow}...")
            result = await session.call_tool("create_post", {
                "content": "Hello from Python MCP client!",
                "platforms": ["linkedin-YOUR_PLATFORM_ID"],
                "scheduledTime": tomorrow
            })
            print(result.content[0].text)

            # Step 3: List scheduled posts
            print("\nListing scheduled posts...")
            posts = await session.call_tool("list_posts", {
                "status": "scheduled"
            })
            print(posts.content[0].text)

asyncio.run(schedule_post_workflow())
```

---

## Programmatic Access (TypeScript)

### Installation

```bash
npm install @modelcontextprotocol/sdk
```

### Basic Example

```typescript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

async function main() {
  const transport = new StreamableHTTPClientTransport(
    new URL("https://mcp.publora.com"),
    {
      headers: {
        Authorization: "Bearer sk_YOUR_API_KEY",
      },
    }
  );

  const client = new Client({
    name: "publora-client",
    version: "1.0.0",
  });

  await client.connect(transport);

  // List tools
  const tools = await client.listTools();
  console.log(`Available tools: ${tools.tools.length}`);

  // Get connections
  const result = await client.callTool({
    name: "list_connections",
    arguments: {},
  });
  console.log(result.content[0]);

  await client.close();
}

main();
```

---

## cURL Testing

Test the MCP server directly with cURL:

### Health Check

```bash
curl https://mcp.publora.com/health
# {"status":"ok","service":"publora-mcp"}
```

### Initialize Session

```bash
curl -X POST https://mcp.publora.com \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer sk_YOUR_API_KEY" \
  -d '{
    "jsonrpc": "2.0",
    "id": 1,
    "method": "initialize",
    "params": {
      "protocolVersion": "2024-11-05",
      "capabilities": {},
      "clientInfo": {
        "name": "curl-test",
        "version": "1.0.0"
      }
    }
  }'
```

### List Tools

```bash
curl -X POST https://mcp.publora.com \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer sk_YOUR_API_KEY" \
  -H "mcp-session-id: YOUR_SESSION_ID" \
  -d '{
    "jsonrpc": "2.0",
    "id": 2,
    "method": "tools/list",
    "params": {}
  }'
```

---

## Environment Variables

For security, store your API key in environment variables:

### Shell

```bash
export PUBLORA_API_KEY="sk_YOUR_API_KEY"
```

### Configuration with env variable

Some clients support environment variable interpolation:

```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer ${PUBLORA_API_KEY}"
      }
    }
  }
}
```

---

## Troubleshooting Setup

### Tools not appearing

1. **Restart your client** — MCP servers load on startup
2. **Check JSON syntax** — Validate at [jsonlint.com](https://jsonlint.com)
3. **Verify type is "http"** — Not "url" or "sse"
4. **Check the URL** — Both `https://mcp.publora.com/mcp` (the address in the Claude directory listing and the server card) and `https://mcp.publora.com` work

### Authentication errors

1. **Include "Bearer" prefix** — `Authorization: Bearer sk_...`
2. **Check key format** — Should start with `sk_`
3. **Generate new key** — At publora.com → **API** in sidebar

> **Note:** Direct REST API calls to `api.publora.com` require `x-publora-key: sk_...`. MCP accepts both headers; use `Authorization: Bearer sk_...` for MCP client configurations unless your client or proxy requires the `x-publora-key` fallback.

### Connection refused

1. **Check internet** — Verify connectivity
2. **Test health endpoint** — `curl https://mcp.publora.com/health`
3. **Check firewall** — Ensure HTTPS is allowed

---

## Next Steps

- [Tools Reference](./tools-reference.md) — All 18 active tools with parameters
- [Examples](./examples.md) — Real-world conversation examples
- [Troubleshooting](./troubleshooting.md) — Common issues and solutions
