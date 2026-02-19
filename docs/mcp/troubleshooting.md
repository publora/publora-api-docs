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

4. **Regenerate your key:**
   - Go to [publora.com](https://publora.com) → Settings → API
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

### "Invalid API key"

**Solutions:**

1. **Check for typos** — Keys start with `sk_`

2. **Ensure no quotes around key in header value:**
   ```json
   // Correct
   "Authorization": "Bearer sk_abc123"

   // Wrong - extra quotes
   "Authorization": "Bearer \"sk_abc123\""
   ```

3. **Regenerate key:**
   - [publora.com](https://publora.com) → Settings → API
   - Generate new key
   - Update all configurations

4. **Verify correct account** — Make sure you're logged into the right Publora account

---

### "Unauthorized" When Calling Tools

**Cause:** API key doesn't have access to the requested resource.

**Solutions:**

1. **Verify workspace** — Check you're connected to the correct Publora workspace

2. **Check subscription** — Ensure your plan includes API access

3. **Verify resource ownership** — You can only access your own workspace's data

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
| Threads | 500 |
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

1. **Verify the ID** — Post IDs look like `pg_abc123`
2. **List posts** — Use `list_posts` to see valid post IDs
3. **Check filters** — The post might have a different status than expected

---

## Session Issues

### "Invalid or missing session"

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

## Rate Limiting

### "Rate limit exceeded"

**Cause:** Too many requests in a short period.

**Solutions:**

1. **Wait and retry** — Rate limits reset after a short period
2. **Reduce request frequency** — Space out your requests
3. **Check your plan** — Higher tiers have higher rate limits

**Rate limit headers:**
```text
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 0
X-RateLimit-Reset: 1709312400
```

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
