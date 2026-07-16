# n8n Integration Guide

Connect Publora to 400+ apps using n8n's HTTP Request node.

## Overview

n8n is a free, self-hostable workflow automation platform. Use the **HTTP Request** node to call the Publora API and automate your social media posting.

## Prerequisites

- n8n instance (cloud or self-hosted)
- Publora API key from [app.publora.com](https://app.publora.com)
- At least one social account connected in Publora

## Setting Up Credentials

### Create Publora Header Auth

1. In n8n, go to **Credentials**
2. Click **Add Credential**
3. Select **Header Auth**
4. Configure:
   - **Name:** Publora API
   - **Header Name:** x-publora-key
   - **Header Value:** YOUR_API_KEY
5. Save

---

## Example 1: Post When Notion Page Created

Automatically share new Notion pages to social media.

### Workflow Setup

1. **Notion Trigger** → Database Item Created
2. **HTTP Request** → POST to Publora

### HTTP Request Configuration

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Method:** `POST`

**Authentication:** Header Auth (Publora API)

**Body (JSON):**
```json
{
  "content": "📝 New article: {{ $json.properties.Name.title[0].plain_text }}\n\nRead it here: {{ $json.url }}",
  "platforms": ["twitter-123456789", "linkedin-ABC123DEF"],
  "scheduledTime": "{{ new Date(Date.now() + 5 * 60 * 1000).toISOString() }}"
}
```

---

## Example 2: Content Calendar from Airtable

Use Airtable as your content calendar and auto-schedule posts.

### Airtable Base Structure

| Content | Platforms | Scheduled Time | Status | Post ID |
|---------|-----------|----------------|--------|---------|
| Monday tip! | twitter-123;linkedin-456 | `<FUTURE_ISO_8601_UTC>` | pending | |

### Workflow Setup

1. **Schedule Trigger** → Every hour
2. **Airtable** → List Records (filter: Status = "pending", Scheduled Time <= now + 1 hour)
3. **Loop Over Items**
4. **Function** → Split platforms string into array
5. **HTTP Request** → POST to Publora
6. **Airtable** → Update Record (set status to "scheduled")

### Function Node: Split Platforms

```javascript
const platforms = $input.first().json.fields.Platforms.split(';').map(p => p.trim());

return [{
  json: {
    ...$input.first().json,
    platformsArray: platforms
  }
}];
```

### HTTP Request Configuration

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Body (JSON):**
```json
{
  "content": "{{ $json.fields.Content }}",
  "platforms": {{ $json.platformsArray }},
  "scheduledTime": "{{ $json.fields['Scheduled Time'] }}"
}
```

---

## Example 3: Auto-Post from RSS Feed

Share new blog posts automatically.

### Workflow Setup

1. **RSS Feed Read** → Check every 15 minutes
2. **IF** → Check if item is new (compare with stored IDs)
3. **HTTP Request** → POST to Publora
4. **Code** → Store processed item IDs

### HTTP Request Configuration

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Body (JSON):**
```json
{
  "content": "🆕 {{ $json.title }}\n\n{{ $json.contentSnippet.substring(0, 200) }}...\n\nRead more: {{ $json.link }}",
  "platforms": ["twitter-123456789", "linkedin-ABC123DEF"],
  "scheduledTime": "{{ new Date(Date.now() + 5 * 60 * 1000).toISOString() }}"
}
```

---

## Example 4: Scheduled Posts from Google Sheets

### Workflow Setup

1. **Google Sheets Trigger** → Row Added
2. **Date & Time** → Parse scheduled time
3. **IF** → Check if scheduled time is in future
4. **HTTP Request** → POST to Publora
5. **Google Sheets** → Update Row (mark as scheduled)

### Code Node: Prepare Post Data

```javascript
const row = $input.first().json;

// Split platforms by semicolon
const platforms = row['Platforms'].split(';').map(p => p.trim());

// Ensure valid ISO 8601 format
const scheduledTime = new Date(row['Scheduled Time']).toISOString();

return [{
  json: {
    content: row['Content'],
    platforms: platforms,
    scheduledTime: scheduledTime
  }
}];
```

### HTTP Request Configuration

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Body (JSON):**
```json
{
  "content": "{{ $json.content }}",
  "platforms": {{ $json.platforms }},
  "scheduledTime": "{{ $json.scheduledTime }}"
}
```

---

## Example 5: Multi-Platform Post with Different Content

Post tailored content to each platform.

### Workflow Setup

1. **Webhook** → Receive trigger with content
2. **Switch** → Branch by platform
3. Multiple **HTTP Request** nodes (one per platform)

### Branch: Twitter (Short form)

```json
{
  "content": "{{ $json.shortContent }}\n\n{{ $json.link }}",
  "platforms": ["twitter-123456789"],
  "scheduledTime": "{{ new Date(Date.now() + 5 * 60 * 1000).toISOString() }}"
}
```

### Branch: LinkedIn (Long form)

```json
{
  "content": "{{ $json.longContent }}\n\n{{ $json.link }}\n\n#business #technology",
  "platforms": ["linkedin-ABC123DEF"],
  "scheduledTime": "{{ new Date(Date.now() + 5 * 60 * 1000).toISOString() }}"
}
```

### Branch: Threads (Medium form)

```json
{
  "content": "{{ $json.mediumContent }}\n\n{{ $json.link }}",
  "platforms": ["threads-987654321"],
  "scheduledTime": "{{ new Date(Date.now() + 5 * 60 * 1000).toISOString() }}"
}
```

---

## Example 6: Upload Image and Post

Upload an image first, then create a post with it.

### Workflow Setup

1. **Trigger** (e.g., Webhook with file URL)
2. **HTTP Request** → Create post to get a postGroupId
3. **HTTP Request** → GET upload URL from Publora (with postGroupId)
4. **HTTP Request** → Download image from source
5. **HTTP Request** → PUT upload to S3 (media auto-attaches via postGroupId)
6. **HTTP Request** → PUT `/update-post/{postGroupId}` to schedule the draft

### Step 2: Create Post

**URL:**
```
https://api.publora.com/api/v1/create-post
```

**Method:** `POST`

**Body:**
```json
{
  "content": "Check out this image!",
  "platforms": ["twitter-123456789"]
}
```

### Step 3: Get Upload URL

**URL:**
```
https://api.publora.com/api/v1/get-upload-url
```

**Method:** `POST`

**Body:**
```json
{
  "fileName": "{{ $json.fileName }}",
  "contentType": "{{ $json.contentType }}",
  "postGroupId": "{{ $node['Create Post'].json.postGroupId }}"
}
```

### Step 5: Upload to S3

**URL:** `{{ $node["Get Upload URL"].json.uploadUrl }}`

**Method:** `PUT`

**Headers:**
```
Content-Type: {{ $json.contentType }}
```

**Body:** Binary data from previous download node

> **Note:** Media is automatically attached to the post via the `postGroupId` provided when requesting the upload URL. No need to pass media references when creating the post.

### Step 6: Schedule the Uploaded Post

**URL:** `https://api.publora.com/api/v1/update-post/{{ $node['Create Post'].json.postGroupId }}`

**Method:** `PUT`

**Body (JSON):**
```json
{
  "status": "scheduled",
  "scheduledTime": "{{ new Date(Date.now() + 5 * 60 * 1000).toISOString() }}"
}
```

The initial create call intentionally makes a draft. Always schedule after the final upload; attaching media to a scheduled group demotes it back to draft.

---

## Example 7: Weekly Content Schedule

Post recurring content every week.

### Workflow Setup

1. **Schedule Trigger** → Every Monday at 9 AM
2. **Code** → Generate weekly content array
3. **Loop Over Items**
4. **Wait** → Stagger posts by day
5. **HTTP Request** → POST to Publora

### Code Node: Generate Weekly Content

```javascript
const baseDate = new Date();
baseDate.setUTCHours(9, 0, 0, 0);

const weeklyContent = [
  { content: 'Monday motivation: Start your week with purpose!', dayOffset: 0 },
  { content: 'Tech tip Tuesday: Version your APIs for stability.', dayOffset: 1 },
  { content: 'Wednesday wisdom: Ship fast, iterate faster.', dayOffset: 2 },
  { content: 'Throwback Thursday: How we reached 10K users.', dayOffset: 3 },
  { content: 'Feature Friday: Check out our new dashboard!', dayOffset: 4 },
];

return weeklyContent.map(item => {
  const scheduledDate = new Date(baseDate);
  scheduledDate.setDate(scheduledDate.getDate() + item.dayOffset);

  return {
    json: {
      content: item.content,
      platforms: ['twitter-123456789', 'linkedin-ABC123DEF'],
      scheduledTime: scheduledDate.toISOString()
    }
  };
});
```

---

## Example 8: Slack Command to Post

Let your team schedule social posts from Slack.

### Workflow Setup

1. **Webhook** → Receive Slack slash command
2. **Code** → Parse command and extract content
3. **HTTP Request** → POST to Publora
4. **HTTP Request** → Respond to Slack

### Code Node: Parse Slack Command

```javascript
const text = $input.first().json.text;

// Parse: /post twitter,linkedin Hello world!
const match = text.match(/^(\S+)\s+(.+)$/);

if (!match) {
  return [{
    json: {
      error: true,
      message: 'Usage: /post twitter,linkedin Your post content'
    }
  }];
}

const platformMap = {
  'twitter': 'twitter-123456789',
  'linkedin': 'linkedin-ABC123DEF',
  'threads': 'threads-987654321'
};

const platforms = match[1].split(',')
  .map(p => platformMap[p.trim().toLowerCase()])
  .filter(Boolean);

return [{
  json: {
    content: match[2],
    platforms: platforms,
    scheduledTime: new Date(Date.now() + 5 * 60 * 1000).toISOString()
  }
}];
```

---

## Error Handling

### Add Error Workflow

1. Create a separate error handling workflow
2. In your main workflow settings, set the error workflow
3. Handle common errors:

```javascript
const statusCode = $input.first().json.statusCode;
const error = $input.first().json.error || $input.first().json.message;

let action = 'retry';
let message = '';

switch (statusCode) {
  case 401:
    action = 'alert';
    message = 'Invalid API key. Check your Publora credentials.';
    break;
  case 403:
    action = 'alert';
    message = 'Access denied: ' + error;
    break;
  case 400:
    action = 'skip';
    message = 'Invalid request: ' + error;
    break;
  case 429:
    action = 'retry';
    message = 'Rate limited. Will retry in 60 seconds.';
    break;
  case 500:
    action = 'retry';
    message = 'Server error. Will retry.';
    break;
  default:
    action = 'alert';
    message = 'Unknown error: ' + error;
}

return [{ json: { action, message, statusCode } }];
```

### Common Error Responses

| Status | Meaning | Solution |
|--------|---------|----------|
| 400 | Bad request (including an unknown platform ID) | Check JSON syntax, required fields, and IDs from platform-connections |
| 401 | Unauthorized | Verify API key in credentials |
| 404 | Resource not found | Check post-group or webhook resource IDs |
| 429 | Rate limited | Add Wait node between requests |

---

## Useful Code Snippets

### Split Platforms String

```javascript
const platforms = $json.platforms.split(';').map(p => p.trim());
return [{ json: { ...($json), platformsArray: platforms } }];
```

### Truncate for Twitter

```javascript
const content = $json.content;
const truncated = content.length > 250
  ? content.substring(0, 247) + '...'
  : content;
return [{ json: { ...($json), twitterContent: truncated } }];
```

### Format Date for Scheduling

```javascript
const dateString = $json.date;
const isoDate = new Date(dateString).toISOString();
return [{ json: { ...($json), scheduledTime: isoDate } }];
```

### Check Response Success

```javascript
const response = $input.first().json;

if (!response.success) {
  throw new Error(response.error || response.message || 'Post creation failed');
}

return [{ json: { postGroupId: response.postGroupId } }];
```

---

## Get Platform Connection IDs

### One-Time Lookup Workflow

1. **Manual Trigger**
2. **HTTP Request** → GET platform connections
3. **Code** → Format output

**HTTP Request:**
```
GET https://api.publora.com/api/v1/platform-connections
```

**Code Node:**
```javascript
const connections = $input.first().json.connections;

console.log('Platform IDs:');
connections.forEach(conn => {
  const platform = conn.platformId.split('-', 1)[0];
  console.log(`  ${platform}: ${conn.platformId} (${conn.username})`);
});

return connections.map(conn => ({ json: conn }));
```

---

## Best Practices

1. **Use Header Auth credential** - Store your API key securely in n8n credentials
2. **Add Wait nodes** - Add 200ms+ delay between API calls to avoid rate limits
3. **Error workflows** - Set up error handling for failed requests
4. **Test first** - Use n8n's manual execution to test before activating
5. **Environment variables** - Store platform IDs in workflow variables
6. **Logging** - Add Code nodes to log important events

### Recommended Workflow Variables

```javascript
// Set in workflow settings
const platformIds = {
  twitter: 'twitter-123456789',
  linkedin: 'linkedin-ABC123DEF',
  threads: 'threads-987654321',
  instagram: 'instagram-456789012'
};
```

---

*[Publora](https://publora.com) — Social media API with free tier, paid plans from $2.99/account*
