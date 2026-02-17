# Zapier Integration Guide

Connect Publora to 5,000+ apps using Zapier's Webhooks feature.

## Overview

While Publora doesn't have a native Zapier app yet, you can easily integrate using Zapier's **Webhooks by Zapier** action to call the Publora API directly.

## Prerequisites

- Zapier account (Free tier works)
- Publora API key from [app.publora.com](https://app.publora.com)
- At least one social account connected in Publora

## Example 1: Post When New Blog Published

Automatically post to social media when you publish a new blog post in WordPress.

### Step 1: Create a New Zap

1. Go to [zapier.com](https://zapier.com) and click "Create Zap"
2. Search for **WordPress** as your trigger app
3. Select **New Post** as the trigger event
4. Connect your WordPress site
5. Test the trigger

### Step 2: Add Webhooks Action

1. Click "+" to add an action
2. Search for **Webhooks by Zapier**
3. Select **POST** as the action event

### Step 3: Configure the Webhook

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Payload Type:** `json`

**Data:**
```json
{
  "content": "New blog post: {{title}} - {{link}}",
  "platforms": ["twitter-YOUR_PLATFORM_ID", "linkedin-YOUR_PLATFORM_ID"]
}
```

**Headers:**
| Key | Value |
|-----|-------|
| `x-publora-key` | `YOUR_API_KEY` |
| `Content-Type` | `application/json` |

### Step 4: Test and Enable

1. Click "Test action" to verify it works
2. Turn on your Zap

---

## Example 2: Schedule Posts from Google Sheets

Use a Google Sheet as your content calendar and automatically schedule posts.

### Google Sheet Format

| A (Content) | B (Platforms) | C (Schedule Time) | D (Posted) |
|-------------|---------------|-------------------|------------|
| Monday motivation! | twitter-123;linkedin-456 | 2026-03-01T09:00:00Z | |
| New feature alert | twitter-123 | 2026-03-01T14:00:00Z | |

### Zap Configuration

**Trigger:** Google Sheets → New Row

**Action:** Webhooks by Zapier → POST

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Data:**
```json
{
  "content": "{{Content}}",
  "platforms": "{{Platforms}}".split(";"),
  "scheduledTime": "{{Schedule Time}}"
}
```

**Note:** For the platforms array, you may need to use Zapier's Formatter to split the semicolon-separated values.

### Using Formatter for Platforms Array

1. Add a **Formatter by Zapier** step before the Webhook
2. Choose **Text** → **Split Text**
3. Input: `{{Platforms}}`
4. Separator: `;`
5. Use the output in your webhook as the platforms value

---

## Example 3: Post RSS Feed Updates

Automatically share new RSS feed items to social media.

### Zap Configuration

**Trigger:** RSS by Zapier → New Item in Feed

**Action:** Webhooks by Zapier → POST

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Data:**
```json
{
  "content": "📰 {{Title}}\n\n{{Description}}\n\nRead more: {{Link}}",
  "platforms": ["twitter-123456789", "linkedin-ABC123DEF"]
}
```

---

## Example 4: Post from Slack Command

Let your team schedule social posts directly from Slack.

### Zap Configuration

**Trigger:** Slack → New Message Posted to Channel

Filter: Message starts with `/post`

**Action 1:** Formatter → Extract text after `/post `

**Action 2:** Webhooks by Zapier → POST

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Data:**
```json
{
  "content": "{{Extracted Text}}",
  "platforms": ["twitter-123456789"]
}
```

---

## Example 5: Weekly Scheduled Post

Post a weekly reminder every Monday at 9 AM.

### Zap Configuration

**Trigger:** Schedule by Zapier → Every Week (Monday at 9 AM)

**Action:** Webhooks by Zapier → POST

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Data:**
```json
{
  "content": "Happy Monday! What are you working on this week? Share in the comments! 👇",
  "platforms": ["twitter-123456789", "linkedin-ABC123DEF", "threads-987654321"]
}
```

---

## Handling Responses

### Success Response

Publora returns:
```json
{
  "success": true,
  "postGroupId": "pg_abc123xyz"
}
```

You can use Zapier's **Paths** or **Filter** to handle this:
- Continue workflow if `success` is `true`
- Send alert if `success` is `false`

### Error Handling

Add a **Paths** step after the webhook:

**Path A:** If `success` equals `true`
- Continue with success actions (e.g., update spreadsheet, send Slack notification)

**Path B:** If `success` does not equal `true`
- Send error notification via email or Slack
- Log to error spreadsheet

---

## Finding Your Platform IDs

To get your platform connection IDs for Zapier:

### Option 1: Use the API

```bash
curl https://api.publora.com/api/v1/platform-connections \
  -H "x-publora-key: YOUR_API_KEY"
```

### Option 2: Zapier Webhook GET

Create a simple Zap:
1. Trigger: Schedule by Zapier → Every Day
2. Action: Webhooks by Zapier → GET
3. URL: `https://api.publora.com/api/v1/platform-connections`
4. Headers: `x-publora-key: YOUR_API_KEY`

Run it once and check the response to get your platform IDs.

---

## Tips and Best Practices

1. **Test First:** Always use Zapier's test feature before enabling a Zap
2. **Rate Limits:** Add a 1-second delay between multiple webhook calls
3. **Error Notifications:** Set up email alerts for failed Zaps
4. **Content Length:** Be mindful of platform character limits (Twitter: 280, LinkedIn: 3000, etc.)
5. **Scheduling:** Use ISO 8601 format for `scheduledTime` (e.g., `2026-03-01T14:00:00.000Z`)

---

## Common Issues

### "Invalid API Key"
- Check that your API key is correctly entered in the headers
- Ensure there are no extra spaces

### "Invalid Platform ID"
- Verify your platform IDs using the GET /platform-connections endpoint
- Platform IDs look like `twitter-123456789` or `linkedin-ABC123DEF`

### "Scheduled time is in the past"
- Make sure your `scheduledTime` is in the future
- Use UTC timezone (ends with `Z`)

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
