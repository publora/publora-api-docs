# Telegram

Publora integrates with Telegram for publishing content to channels and groups via bot connection. Telegram supports rich text formatting with markdown syntax.

## Platform ID Format

```
telegram-{chatId}
```

Where `{chatId}` is your Telegram channel or group chat ID assigned during bot connection setup.

## Requirements

- A Telegram bot created via [@BotFather](https://t.me/BotFather)
- The bot's token and target channel/group name provided during setup
- The bot must be added as an **administrator** of the target channel or group
- API key from Publora

## Connection Setup

1. Create a bot with [@BotFather](https://t.me/BotFather) on Telegram
2. Copy the bot token provided by BotFather
3. Add the bot as an administrator to your target channel or group
4. In the Publora dashboard, provide the bot token and channel name (e.g., `@mychannel`)
5. Publora will verify the connection and assign a platform ID

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 1,024 characters (bot API), 4,096 characters (MTProto) |
| Images | Yes | JPEG (WebP auto-converted) |
| Videos | Yes | MP4 format |
| Formatting | Yes | Bold, italic, code, links, blockquotes |

## Text Formatting

Telegram supports markdown-style formatting in message text. You can use the following syntax in your `content`:

| Syntax | Result | Example |
|--------|--------|---------|
| `**text**` | **Bold** | `**important**` |
| `_text_` | _Italic_ | `_emphasis_` |
| `` `code` `` | `Inline code` | `` `variable` `` |
| ```` ```code``` ```` | Code block | ```` ```print("hello")``` ```` |
| `[text](url)` | Hyperlink | `[click here](https://example.com)` |
| `> text` | Blockquote | `> quoted text` |

## Caption vs. Message

When posting with media (images or videos), the text content is sent as a **caption** attached to the media. Captions have a stricter limit of 1,024 characters when using the bot API. Text-only messages support up to 4,096 characters via MTProto.

## Examples

### Post a Text Message

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: '**Product Update v2.5**\n\nWe have shipped the following improvements:\n\n- _Faster API response times_ (avg 45ms)\n- New `batch` endpoint for bulk operations\n- Improved error messages with `error_code` field\n\n> This update is backward compatible. No migration needed.\n\nFull changelog: [docs.example.com/changelog](https://docs.example.com/changelog)',
    platforms: ['telegram-1001234567890']
  })
});

const data = await response.json();
console.log(data);
```

**Python (requests)**

```python
import requests

content = """**Product Update v2.5**

We have shipped the following improvements:

- _Faster API response times_ (avg 45ms)
- New `batch` endpoint for bulk operations
- Improved error messages with `error_code` field

> This update is backward compatible. No migration needed.

Full changelog: [docs.example.com/changelog](https://docs.example.com/changelog)"""

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': content,
        'platforms': ['telegram-1001234567890']
    }
)

data = response.json()
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "**Product Update v2.5**\n\nWe have shipped the following improvements:\n\n- _Faster API response times_ (avg 45ms)\n- New `batch` endpoint for bulk operations\n- Improved error messages with `error_code` field\n\n> This update is backward compatible. No migration needed.\n\nFull changelog: [docs.example.com/changelog](https://docs.example.com/changelog)",
    "platforms": ["telegram-1001234567890"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const content = `**Product Update v2.5**

We have shipped the following improvements:

- _Faster API response times_ (avg 45ms)
- New \`batch\` endpoint for bulk operations
- Improved error messages with \`error_code\` field

> This update is backward compatible. No migration needed.

Full changelog: [docs.example.com/changelog](https://docs.example.com/changelog)`;

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content,
  platforms: ['telegram-1001234567890']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with an Image

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: '**New Dashboard Preview**\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.',
    platforms: ['telegram-1001234567890'],
    mediaUrls: ['https://example.com/images/dashboard-preview.jpeg']
  })
});

const data = await response.json();
console.log(data);
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': '**New Dashboard Preview**\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.',
        'platforms': ['telegram-1001234567890'],
        'mediaUrls': ['https://example.com/images/dashboard-preview.jpeg']
    }
)

data = response.json()
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "**New Dashboard Preview**\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.",
    "platforms": ["telegram-1001234567890"],
    "mediaUrls": ["https://example.com/images/dashboard-preview.jpeg"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: '**New Dashboard Preview**\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.',
  platforms: ['telegram-1001234567890'],
  mediaUrls: ['https://example.com/images/dashboard-preview.jpeg']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

### Post with a Video

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: '**Feature Demo**\n\nWatch our 60-second demo of the new collaboration tools.',
    platforms: ['telegram-1001234567890'],
    mediaUrls: ['https://example.com/videos/collab-demo.mp4']
  })
});

const data = await response.json();
console.log(data);
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': '**Feature Demo**\n\nWatch our 60-second demo of the new collaboration tools.',
        'platforms': ['telegram-1001234567890'],
        'mediaUrls': ['https://example.com/videos/collab-demo.mp4']
    }
)

data = response.json()
print(data)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "**Feature Demo**\n\nWatch our 60-second demo of the new collaboration tools.",
    "platforms": ["telegram-1001234567890"],
    "mediaUrls": ["https://example.com/videos/collab-demo.mp4"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: '**Feature Demo**\n\nWatch our 60-second demo of the new collaboration tools.',
  platforms: ['telegram-1001234567890'],
  mediaUrls: ['https://example.com/videos/collab-demo.mp4']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

## Platform Quirks

- **Bot must be admin**: The Telegram bot must be added as an administrator to the target channel or group before Publora can post. Without admin permissions, posts will fail.
- **Caption character limit**: When posting with media (images or videos), the text is sent as a caption with a limit of 1,024 characters via the bot API. Text-only messages support up to 4,096 characters via MTProto.
- **WebP auto-conversion**: WebP images are automatically converted to JPEG before sending to Telegram.
- **Markdown formatting**: Telegram supports a subset of markdown. Use `**bold**`, `_italic_`, `` `code` ``, `[text](url)`, and `> blockquote`. Not all markdown features are supported (e.g., headers are not rendered as headers).
- **Channel name format**: When setting up the connection, provide the channel name with the `@` prefix (e.g., `@mychannel`). Private channels require the numeric chat ID instead.
- **Bot token security**: Your bot token is stored securely by Publora and is never exposed in API responses. Treat your bot token like a password.
- **Silent messages**: Telegram supports silent message delivery, but this feature is not currently available through the Publora API.
- **Message editing**: Once posted, Telegram messages can be edited, but this capability is not currently exposed through the Publora API.

## Character Limits

| Element | Limit |
|---------|-------|
| Text message (MTProto) | 4,096 characters |
| Media caption (Bot API) | 1,024 characters |
| Bot username | 32 characters |
| Channel name | 128 characters |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
