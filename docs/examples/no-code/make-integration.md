# Make (Integromat) Integration Guide

Connect Publora to 1,000+ apps using Make's HTTP module.

## Overview

Make (formerly Integromat) provides powerful visual workflow automation. Use the **HTTP** module to call the Publora API and automate your social media posting.

## Prerequisites

- Make account (Free tier includes 1,000 operations/month)
- Publora API key from [app.publora.com](https://app.publora.com)
- At least one social account connected in Publora

## Setting Up the HTTP Module

### Create a Publora Connection Template

1. In Make, go to **Connections** in the left sidebar
2. Click **Add** and select **HTTP**
3. Name it "Publora API"
4. Save for reuse in all your scenarios

---

## Example 1: Post When Notion Page Created

Automatically share new Notion pages to social media.

### Scenario Setup

1. Create a new scenario
2. Add **Notion** → Watch Database Items
3. Add **HTTP** → Make a request

### HTTP Module Configuration

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Method:** `POST`

**Headers:**

| Name | Value |
|------|-------|
| `x-publora-key` | `YOUR_API_KEY` |
| `Content-Type` | `application/json` |

**Body type:** `Raw`

**Content type:** `JSON (application/json)`

**Request content:**
```json
{
  "content": "📝 New article: {{1.properties.Name.title[].plain_text}}\n\nRead it here: {{1.url}}",
  "platforms": ["twitter-123456789", "linkedin-ABC123DEF"],
  "scheduledTime": "{{formatDate(addMinutes(now; 5); \"YYYY-MM-DDTHH:mm:ss.SSS[Z]\")}}"
}
```

---

## Example 2: Content Calendar from Airtable

Use Airtable as your content calendar and auto-schedule posts.

### Airtable Base Structure

| Content | Platforms | Scheduled Time | Status | Post ID |
|---------|-----------|----------------|--------|---------|
| Monday tip! | twitter-123;linkedin-456 | `<FUTURE_ISO_8601_UTC>` | pending | |

### Scenario Setup

1. **Airtable** → Search Records (filter: Status = "pending")
2. **Iterator** → Iterate through records
3. **Tools** → Set Variable (split platforms by semicolon)
4. **HTTP** → Make a request (to Publora)
5. **Airtable** → Update Record (set status to "scheduled", save post ID)

### HTTP Configuration

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Method:** `POST`

**Headers:**
```
x-publora-key: YOUR_API_KEY
Content-Type: application/json
```

**Request content:**
```json
{
  "content": "{{2.fields.Content}}",
  "platforms": {{3.value}},
  "scheduledTime": "{{2.fields.`Scheduled Time`}}"
}
```

### Update Airtable After Success

In the Airtable Update Record module:
- **Status:** `scheduled`
- **Post ID:** `{{4.data.postGroupId}}`

---

## Example 3: Auto-Post from RSS Feed

Share new blog posts automatically.

### Scenario Setup

1. **RSS** → Watch RSS feed items
2. **Text Parser** → HTML to text (clean description)
3. **HTTP** → Make a request

### HTTP Configuration

**Request content:**
```json
{
  "content": "🆕 {{1.title}}\n\n{{2.text}}\n\nRead more: {{1.link}}",
  "platforms": ["twitter-123456789", "linkedin-ABC123DEF"],
  "scheduledTime": "{{formatDate(addMinutes(now; 5); \"YYYY-MM-DDTHH:mm:ss.SSS[Z]\")}}"
}
```

---

## Example 4: Schedule Posts from Google Sheets

### Scenario Setup

1. **Google Sheets** → Watch Rows (new rows only)
2. **Router** → Check if scheduled time is in the future
3. **HTTP** → Make a request
4. **Google Sheets** → Update Row (mark as posted)

### HTTP Configuration

**Request content:**
```json
{
  "content": "{{1.`Column A`}}",
  "platforms": {{split(1.`Column B`; ";")}},
  "scheduledTime": "{{1.`Column C`}}"
}
```

---

## Example 5: Multi-Platform Post with Different Content

Post tailored content to each platform.

### Scenario Setup

1. **Trigger** (any source)
2. **Router** → Branch for each platform
3. Multiple **HTTP** modules (one per platform)

### Branch 1: Twitter (280 chars)

```json
{
  "content": "{{substring(1.content; 0; 250)}}... {{1.link}}",
  "platforms": ["twitter-123456789"],
  "scheduledTime": "{{formatDate(addMinutes(now; 5); \"YYYY-MM-DDTHH:mm:ss.SSS[Z]\")}}"
}
```

### Branch 2: LinkedIn (longer form)

```json
{
  "content": "{{1.full_content}}\n\n{{1.link}}\n\n#business #technology",
  "platforms": ["linkedin-ABC123DEF"],
  "scheduledTime": "{{formatDate(addMinutes(now; 5); \"YYYY-MM-DDTHH:mm:ss.SSS[Z]\")}}"
}
```

### Branch 3: Threads (500 chars)

```json
{
  "content": "{{substring(1.content; 0; 470)}}...\n\n{{1.link}}",
  "platforms": ["threads-987654321"],
  "scheduledTime": "{{formatDate(addMinutes(now; 5); \"YYYY-MM-DDTHH:mm:ss.SSS[Z]\")}}"
}
```

---

## Example 6: Upload Image and Post

Upload an image first, then create a post with it.

### Scenario Setup

1. **Trigger** (e.g., Google Drive → Watch Files)
2. **HTTP** → Create post to get a postGroupId
3. **HTTP** → Get upload URL from Publora (with postGroupId)
4. **HTTP** → Upload file to S3 (media auto-attaches via postGroupId)
5. **HTTP** → Update post to `scheduled` after the upload

### Step 2: Create Post

**URL:** `https://api.publora.com/api/v1/create-post`

**Method:** `POST`

**Request content:**
```json
{
  "content": "Check out this image!",
  "platforms": ["twitter-123456789"]
}
```

### Step 3: Get Upload URL

**URL:** `https://api.publora.com/api/v1/get-upload-url`

**Method:** `POST`

**Request content:**
```json
{
  "fileName": "{{1.name}}",
  "contentType": "{{1.mimeType}}",
  "postGroupId": "{{2.data.postGroupId}}"
}
```

### Step 4: Upload to S3

**URL:** `{{3.data.uploadUrl}}`

**Method:** `PUT`

**Headers:**
```
Content-Type: {{1.mimeType}}
```

**Body type:** `Binary`

**Content:** `{{1.data}}`

> **Note:** Media is automatically attached to the post via the `postGroupId` provided when requesting the upload URL. No need to pass media references when creating the post.

### Step 5: Schedule the Uploaded Post

**URL:** `https://api.publora.com/api/v1/update-post/{{2.data.postGroupId}}`

**Method:** `PUT`

**Request content:**
```json
{
  "status": "scheduled",
  "scheduledTime": "{{formatDate(addMinutes(now; 5); \"YYYY-MM-DDTHH:mm:ss.SSS[Z]\")}}"
}
```

The create step intentionally makes a draft. Schedule only after the final upload; a media attachment demotes an already-scheduled group to draft.

---

## Error Handling

### Add Error Handler

1. Right-click on the HTTP module
2. Select **Add error handler**
3. Choose **Resume** or **Rollback** based on your needs

### Common Error Responses

| Status | Meaning | Solution |
|--------|---------|----------|
| 400 | Bad request (including an unknown platform ID) | Check JSON syntax, required fields, and IDs from platform-connections |
| 401 | Unauthorized | Verify API key |
| 404 | Resource not found | Check post-group or webhook resource IDs |
| 429 | Rate limited | Add delay between requests |

### Retry on Failure

Use Make's built-in retry:
1. Click on HTTP module settings (gear icon)
2. Enable **Retry on failure**
3. Set **Max retries:** 3
4. Set **Interval:** 5 seconds

---

## Useful Functions

### Split Platforms String

Convert `"twitter-123;linkedin-456"` to array:
```
{{split(1.platforms; ";")}}
```

### Truncate for Twitter

For compatibility with standard X accounts, keep content under 280 characters:
```
{{if(length(1.content) > 250; substring(1.content; 0; 247) + "..."; 1.content)}}
```

### Format Date for Scheduling

Convert to ISO 8601:
```
{{formatDate(1.date; "YYYY-MM-DDTHH:mm:ss.000[Z]")}}
```

### Add Delay Between Requests

Use **Sleep** module between HTTP calls:
```
300 (milliseconds)
```

---

## Webhook Trigger (Incoming)

You can also trigger Make scenarios from external sources:

1. Add **Webhooks** → Custom webhook as trigger
2. Copy the webhook URL
3. Use this URL in your app to trigger the scenario

### Example: Trigger from Your App

```bash
curl -X POST "https://hook.make.com/your-webhook-id" \
  -H "Content-Type: application/json" \
  -d '{"content": "Post this!", "platforms": ["twitter-123456789"]}'
```

---

## Best Practices

1. **Use Variables:** Store your API key and platform IDs as scenario variables
2. **Add Delays:** Use Sleep module (300ms) between multiple API calls
3. **Error Alerts:** Set up email notifications for scenario failures
4. **Test Mode:** Always test with a single item before running full scenarios
5. **Scheduling:** Run posting scenarios during business hours
6. **Logging:** Use Make's execution history to debug issues

---

*[Publora](https://publora.com) — Social media API with free tier, paid plans from $2.99/account*
