# MCP Troubleshooting Guide

Common issues and solutions for Publora MCP Server.

## Connection Issues

### "API key required" Error

**Cause:** Missing or malformed Authorization header.

**Solutions:**

1. **Check header format:**
   ```json
   {
     "headers": {
       "Authorization": "Bearer sk_YOUR_API_KEY"
     }
   }
   ```

2. **Ensure "Bearer" prefix is included:**
   ```text
   Correct: "Bearer sk_abc123..."
   Wrong: "sk_abc123..."
   ```

3. **Verify no extra spaces:**
   ```text
   Correct: "Bearer sk_abc123"
   Wrong: "Bearer  sk_abc123" (double space)
   Wrong: " Bearer sk_abc123" (leading space)
   ```

4. **Generate a new key:**
   - Go to [publora.com](https://publora.com) → **API** in the sidebar
   - Click "Generate New Key"
   - Update your configuration

---

### Tools Not Showing in AI Client

**Cause:** MCP server not loaded properly.

**Solution 1: Restart your AI client**

MCP servers load on startup. Close and reopen your client completely.

**Solution 2: Check config file location**

| Client | Config Location |
|--------|-----------------|
| Claude Code | `~/.claude.json` or `.mcp.json` in project |
| Cursor | `~/.cursor/mcp.json` or `.cursor/mcp.json` in project |
| Claude Desktop | Settings → Developer → Edit Config |

**Solution 3: Validate JSON syntax**

Use [jsonlint.com](https://jsonlint.com) to check for errors.

Common JSON mistakes:
```json
// Wrong - trailing comma
{
  "mcpServers": {
    "publora": {
      "type": "http",
    }
  }
}

// Wrong - missing quotes
{
  mcpServers: {
    publora: {}
  }
}

// Wrong - single quotes
{
  'mcpServers': {
    'publora': {}
  }
}
```

**Solution 4: Verify config structure**

Correct structure:
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

**Solution 5: Check with Claude Code**

Run `/mcp` to see connected servers and their status.

---

### "Connection refused" or Network Error

**Cause:** Cannot reach mcp.publora.com.

**Solution 1: Check internet connection**

```bash
ping google.com
```

**Solution 2: Verify server is running**

```bash
curl https://mcp.publora.com/health
```

Expected response:
```json
{"status":"ok","service":"publora-mcp"}
```

**Solution 3: Check for firewall/proxy**

- Ensure HTTPS (port 443) is allowed
- Check corporate proxy settings
- Try from a different network

**Solution 4: Check DNS resolution**

```bash
nslookup mcp.publora.com
```

---

## Authentication Issues

### "API key required" / "Malformed API key" (401)

The MCP server returns one of these two 401 responses if the request is missing a usable API key:

| Response `error` | Cause |
|---|---|
| `API key required. Use Authorization: Bearer sk_<your_key> or x-publora-key header.` | No `Authorization` or `x-publora-key` header at all. |
| `Malformed API key. Publora keys start with sk_ and are 20-200 characters long.` | A key was sent, but it doesn't match the expected shape (e.g. you pasted the placeholder `sk_YOUR_API_KEY`, or the value is truncated). |

Both responses include a `docs` field in the JSON body. The authentication challenge depends on the deployment and failure:

| Request/deployment | `WWW-Authenticate` |
|---|---|
| Missing key, OAuth-enabled deployment (including public `mcp.publora.com`) | Bearer challenge pointing to the protected-resource metadata |
| Missing key, static-key-only deployment | Absent |
| Malformed key, either mode | Absent |
| Well-formed but invalid key, either mode | Absent |

Inspect the JSON body with:

```bash
curl -v -X POST https://mcp.publora.com \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer sk_YOUR_API_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{}}'
```

**Solutions:**

1. **Check for typos** — Keys start with `sk_` and are 20-200 characters long.

2. **Ensure no quotes around the key in the header value:**
   ```json
   // Correct
   "Authorization": "Bearer sk_abc123..."

   // Wrong - extra quotes
   "Authorization": "Bearer \"sk_abc123...\""
   ```

3. **Use the fallback header if your client / proxy strips `Authorization`** — the server also accepts `x-publora-key: sk_<your_key>`. Some Cloudflare / reverse-proxy configurations strip the `Authorization` header, so this is a useful alternative.

4. **Generate a new key:**
   - [publora.com](https://publora.com) → **API** in the sidebar
   - Click **Generate API Key**
   - Update all configurations

5. **Verify correct account** — Make sure you're logged into the right Publora account.

---

### "Unauthorized" When Calling Tools

**Cause:** API key doesn't have access to the requested resource.

**Solutions:**

1. **Verify workspace** — Check you're connected to the correct Publora workspace

2. **Check subscription** — Ensure your plan includes API access

3. **Verify resource ownership** — You can only access your own workspace's data

---

### OAuth / claude.ai connector

**`mcp.publora.com` supports OAuth 2.1** (Dynamic Client Registration + PKCE) in addition to static API keys. The claude.ai **custom connector** uses it automatically:

1. claude.ai → **Settings → Connectors → Add custom connector** → URL `https://mcp.publora.com`.
2. Click **Connect** → a **"Connect Publora to Claude"** consent page opens.
3. Paste your `sk_...` API key and **Authorize**. (Your API key *is* the credential — the OAuth access token wraps it; Publora stores no extra secret.)

**If you instead see** `{"error":"oauth_not_supported"}` **(404) on `/register` or `/.well-known/oauth-*`:** you are hitting a deployment running in **static-key-only mode** (OAuth disabled). The public `mcp.publora.com` has OAuth enabled; a self-hosted/older instance without the OAuth signing secret does not. On such an instance, disable OAuth in your client and send a static key header (`Authorization: Bearer sk_...` or `x-publora-key: sk_...`).

**Headless CLIs (mcporter, custom agents):** the consent step is an interactive web page, so non-interactive clients can't complete it — use a static API-key header instead of the OAuth flow.

---

## Tool Errors

### "Platform not found"

**Cause:** Using an invalid platform ID.

**Solution:**

1. First call `list_connections` to get valid platform IDs
2. Platform IDs look like: `twitter-123456`, `linkedin-abc123`
3. Don't use generic names like "twitter" — use the full ID

```text
You: "List my connections"
Claude: You have these platforms:
  - linkedin-abc123
  - twitter-xyz789

You: "Schedule to linkedin-abc123"  // Correct
You: "Schedule to linkedin"          // Wrong
```

---

### "Invalid scheduled time"

**Cause:** Datetime format is wrong.

**Solution:**

Use ISO 8601 format: `2026-03-01T14:00:00Z`

**Correct formats:**
```text
2026-03-01T14:00:00Z           (UTC)
2026-03-01T09:00:00-05:00      (with timezone offset)
2026-03-01T14:00:00.000Z       (with milliseconds)
```

**Wrong formats:**
```text
March 1, 2026
03/01/2026 2pm
2026-03-01 14:00
tomorrow at 9am (AI converts this, but tool needs ISO 8601)
```

---

### "Post content too long"

**Cause:** Content exceeds platform character limits.

**Platform limits:**

| Platform | Character Limit |
|----------|-----------------|
| Twitter/X | 280 |
| LinkedIn | 3,000 |
| Instagram | 2,200 |
| Threads | 500 (10,000 with text attachment) |
| Bluesky | 300 |
| Mastodon | 500 |
| Telegram | 4,096 |
| Facebook | 63,206 |
| TikTok | 2,200 |
| YouTube | 5,000 |

**Solution:** Shorten your content or use a platform with higher limits.

---

### "Post group not found"

**Cause:** Invalid post group ID or post was deleted.

**Solutions:**

1. **Verify the ID** — Post IDs are MongoDB ObjectIds (e.g., `67a1b2c3d4e5f6a7b8c9d0e1`)
2. **List posts** — Use `list_posts` to see valid post IDs
3. **Check filters** — The post might have a different status than expected

---

## Session Issues

### "Invalid or missing session. Send a POST without mcp-session-id to start."

**Cause:** MCP session expired or not initialized.

**Solution:**

This usually resolves automatically. The server creates a new session on the next request.

If persistent:

1. **Restart your AI client**
2. **Check your API key is still valid**
3. **Verify server is responding:**
   ```bash
   curl https://mcp.publora.com/health
   ```

---

## Configuration Mistakes

### Common Configuration Errors

**Wrong - missing "type":**
```json
{
  "mcpServers": {
    "publora": {
      "url": "https://mcp.publora.com"
    }
  }
}
```

**Wrong - "url" type instead of "http":**
```json
{
  "mcpServers": {
    "publora": {
      "type": "url",
      "url": "https://mcp.publora.com"
    }
  }
}
```

**Wrong - missing Bearer prefix:**
```json
{
  "headers": {
    "Authorization": "sk_abc123..."
  }
}
```

**Wrong - trailing slash in URL:**
```json
{
  "url": "https://mcp.publora.com/"
}
```

**Correct configuration:**
```json
{
  "mcpServers": {
    "publora": {
      "type": "http",
      "url": "https://mcp.publora.com",
      "headers": {
        "Authorization": "Bearer sk_abc123..."
      }
    }
  }
}
```

---

## Plan Limit Errors

### "Post limit reached" / "Scheduled post limit reached"

**Cause:** You have exceeded a plan-based limit (monthly posts, connections, scheduled posts, or schedule horizon).

**Note:** MCP surfaces the full API error in the tool exception. Match stable codes such as `POST_LIMIT_REACHED` or `SCHEDULED_POST_LIMIT_REACHED`; the corresponding API `error` values are `"Post limit reached"` and `"Scheduled post limit reached"`.

**Solutions:**

1. **Check your plan limits:**
   - **Starter (free):** 15 posts/month (account-wide), up to 3 connected accounts. **Includes** API and MCP access (`apiAccess: true`, `mcpAccess: true`) and can post to any of the 10 platforms.
   - **Pro:** 100 posts/month per connection
   - **Premium:** 500 posts/month per connection
2. **Upgrade your plan** — Higher tiers have higher limits
3. **Wait for the monthly reset** — Monthly post counts reset at the start of each billing cycle

---

## mcporter / OpenClaw Issues

### "Unknown MCP server" or "auth required" Error

**Cause:** mcporter expects the Claude Desktop / Cursor-compatible top-level config key `mcpServers` (not `servers`). With the wrong key, the file fails schema validation and mcporter falls back to OAuth. Publora MCP supports OAuth, but its consent page is interactive, so a headless mcporter process cannot complete that flow.

**Solution 1: Use the correct config file shape**

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

Then run:
```bash
mcporter list --config config/mcporter.json
```

If `mcporter list` still reports `auth required` even with the Bearer header in this config, mcporter is merging another config source (e.g. `~/.claude.json`, `~/.mcporter/`). Run with `--verbose` to see which file is supplying the entry:

```bash
mcporter list --config config/mcporter.json --verbose
```

> **Don't run `mcporter auth publora`.** `mcp.publora.com` *does* support OAuth 2.1 (DCR + PKCE), but its consent step is an **interactive** "paste your API key" web page that a headless CLI can't complete — so authenticate mcporter with a static key header (`Authorization: Bearer sk_...` or `x-publora-key`) instead. (The OAuth flow is for the claude.ai web connector.)

**Solution 2: Check mcporter version**

Update to the latest version:
```bash
npm install -g mcporter
```

**Solution 3: Test connection manually**

```bash
curl -X POST https://mcp.publora.com \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer sk_YOUR_API_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0.0"}}}'
```

**Note:** The `Accept` header is technically required by the MCP Streamable HTTP spec, but our server auto-fixes missing headers for compatibility with clients that don't send them.

---

## Debugging Tips

### Enable Verbose Logging

**Claude Code:**
```bash
CLAUDE_DEBUG=1 claude
```

**Python client:**
```python
import logging
logging.basicConfig(level=logging.DEBUG)
```

### Test with cURL

**Health check:**
```bash
curl -v https://mcp.publora.com/health
```

**Test authentication:**
```bash
curl -X POST https://mcp.publora.com \
  -H "Content-Type: application/json" \
  -H "Accept: application/json, text/event-stream" \
  -H "Authorization: Bearer sk_YOUR_API_KEY" \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2024-11-05","capabilities":{},"clientInfo":{"name":"test","version":"1.0.0"}}}'
```

### Check Tool Response

When a tool fails, check the response content:

```python
result = await session.call_tool("list_posts", {})
print(result.content[0].text)  # Check for error messages
```

---

## Getting Help

If you're still stuck:

1. **Documentation:** Check [docs.publora.com](https://docs.publora.com)
2. **Email:** serge@publora.com
3. **Twitter/X:** [@publorainc](https://x.com/publorainc)

**When reporting issues, include:**

- Your AI client (Claude Code, Cursor, etc.) and version
- Config file (with API key redacted)
- Full error message
- Steps to reproduce
- Output of `curl https://mcp.publora.com/health`
