# Telegram API - Post to Channel via REST API

Post to Telegram channels and groups programmatically using the Publora REST API. A simpler alternative to the Telegram Bot API, Telethon, Pyrogram, or MTProto libraries.

## Telegram API Overview

Publora provides a unified REST API for publishing content to Telegram channels and groups via bot or MTProto connection. Supports rich text formatting with markdown syntax, images, and videos. No need to manage Telegram bot tokens directly, handle MTProto complexity, or implement message queuing. Publora supports two connection types: **bot** (using a BotFather token) and **mtproto** (using a user session), each with different capabilities and limits.

### Why Use Publora Instead of Telegram Bot API / Telethon / Pyrogram?

| Feature | Publora API | Telegram Bot API / Telethon |
|---------|-------------|----------------------------|
| Authentication | Single API key | Bot token + channel setup |
| Message formatting | Markdown support | Markdown/HTML support |
| Media handling | Automatic | Manual upload |
| Multi-platform | Post to 10 platforms | Telegram only |
| Setup time | 5 minutes | 15-30 minutes |
| Rate limiting | Handled | Manual implementation |

### Keywords: Telegram API, Telegram Bot API, Telegram channel API, Telegram posting API, post to Telegram programmatically, Telegram REST API, Telegram developer API, Telegram automation API, Telegram message API, send message Telegram API, Telethon alternative, Pyrogram alternative, MTProto API

## Platform ID Format

```
telegram-{chatId}
```

Where `{chatId}` is your Telegram channel or group chat ID assigned during bot connection setup. Bot API chat IDs of channels and supergroups are negative and start with `-100`, so a bot connection's platform ID has a double dash, for example `telegram--1001234567890`. MTProto connections store the channel's own positive ID, so their platform ID has a single dash, for example `telegram-1234567890`. Copy `platformId` verbatim from [`GET /platform-connections`](../endpoints/platform-connections.md) instead of building it.

## Requirements

- A Telegram bot created via [@BotFather](https://t.me/BotFather)
- The bot's token and the target channel, identified during setup by its public `@username` or, for a private channel, by its `-100…` chat ID or a `t.me/c/…` message link
- The bot must be added as an **administrator** of the target channel or supergroup. In a channel, the **`can_post_messages`** (Post Messages) permission must be explicitly enabled; being an admin alone is not enough. Regular (basic) groups are not supported; convert the group to a supergroup or use a channel
- API key from Publora

## Connection Types

Publora supports two connection types for Telegram:

### Bot Connection (Default)

1. Create a bot with [@BotFather](https://t.me/BotFather) on Telegram
2. Copy the bot token provided by BotFather
3. Add the bot as an administrator to your target channel or supergroup
4. In the Publora dashboard, provide the bot token and identify the channel:
   - **Public channel:** its username, e.g. `@mychannel`, `mychannel`, or `t.me/mychannel`
   - **Private channel** (no public username): its chat ID, e.g. `-1001234567890`, or a link to any message in the channel (choose **Copy Link** on the message), e.g. `https://t.me/c/1234567890/42`. The number after `t.me/c/` is the chat ID without its `-100` prefix.
5. Publora will verify the connection and assign a platform ID

Invite links (`t.me/+…`, `t.me/joinchat/…`) are rejected: the Bot API cannot identify a channel from an invite link. Use the chat ID or a message link instead.

A channel without a public username is saved with `username: null`, and `displayName` holds the channel title. [`GET /platform-connections`](../endpoints/platform-connections.md) and the MCP `list_connections` tool return both fields; [`test-connection`](../endpoints/test-connection.md) returns `username: null` and has no `displayName`.

### MTProto Connection

MTProto connections use a Telegram **user session** instead of a bot, enabling higher limits (2,048-character captions with Premium, 4 GB file uploads). MTProto requires a **Telegram Premium** subscription on the service user account. Note: Publora's code enforces a 4,096-character limit for all text fields, but Telegram's actual caption limit on media posts is 1,024 characters (regular users) or 2,048 characters (Premium). The 4,096 limit applies only to text-only messages. Long captions on media posts may be truncated or rejected by Telegram's API.

**Service user concept:** The MTProto connection operates through a dedicated Telegram user account (the "service user") that acts on behalf of your organization. This user must be an admin of the target channel/group. Because the service user is a regular Telegram account (not a bot), it benefits from the higher user-tier limits. The service user's session is stored encrypted by Publora in the connection record and decrypted at runtime.

> **Note:** MTProto connections require the service user to have an active Telegram Premium subscription. Without Premium, certain features (such as the extended 2,048-character caption limit on media posts) will not be available. For self-hosted deployments, the `TELEGRAM_MT_ALLOW_NON_PREMIUM` environment variable can be set to bypass the Premium requirement for MTProto connections.

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 4,096 characters (text-only messages, both bot and MTProto); 1,024 characters (captions with media, bot API) |
| Images | Yes | JPEG, PNG, GIF, BMP, WebP (WebP auto-converted to JPEG) |
| Videos | Yes | MP4, MOV, AVI, MKV, WebM |
| Formatting | Yes | Bold, italic, code, links, blockquotes |

## Text Formatting

Telegram supports markdown-style formatting in message text. You can use the following syntax in your `content`:

| Syntax | Result | Example |
|--------|--------|---------|
| `*text*` | **Bold** | `*important*` |
| `_text_` | _Italic_ | `_emphasis_` |
| `` `code` `` | `Inline code` | `` `variable` `` |
| ```` ```code``` ```` | Code block | ```` ```print("hello")``` ```` |
| `[text](url)` | Hyperlink | `[click here](https://example.com)` |
| `> text` | Blockquote | `> quoted text` |

## Caption vs. Message

When posting with media (images or videos), the text content is sent as a **caption** attached to the media. Captions have a stricter limit of 1,024 characters when using the bot API. MTProto with Premium supports up to 2,048-character captions on media posts (not 4,096 -- the 4,096 limit applies only to text-only messages). Text-only messages (without media) support up to 4,096 characters for both bot API and MTProto connections.

## Telegram Post Options

Publora supports Telegram-specific post options that control message delivery and content protection. These options are available in the Publora dashboard under "Telegram Presets" when creating or editing a post with a Telegram channel selected.

### Available Options

| Option | Description | API Parameter |
|--------|-------------|---------------|
| **Disable link preview** | Prevents Telegram from generating link previews for URLs in the message | `disableWebPagePreview` |
| **Silent notification** | Sends the message without triggering a notification sound on recipients' devices | `disableNotification` |
| **Protect content** | Prevents recipients from forwarding, saving, or copying the message content | `protectContent` |

"Caption above media" is not currently configurable. `showCaptionAboveMedia` is a latent read in the internal MTProto publisher, but there is no dashboard setter, persisted post-group field, or reachable product flow that supplies it. It is not an accepted REST `platformSettings.telegram` key; sending it through the API returns `400 PLATFORM_SETTING_UNKNOWN`.

### Using Post Options via API

To set Telegram-specific options via the API, include the `platformSettings.telegram` object in your request:

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: '*Exclusive Update*\n\nThis content is protected and cannot be forwarded.',
    platforms: ['telegram--1001234567890'],
    platformSettings: {
      telegram: {
        disableNotification: true,      // Send silently
        disableWebPagePreview: true,    // No link previews
        protectContent: true            // Prevent forwarding/saving
      }
    }
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
        'content': '*Exclusive Update*\n\nThis content is protected and cannot be forwarded.',
        'platforms': ['telegram--1001234567890'],
        'platformSettings': {
            'telegram': {
                'disableNotification': True,
                'disableWebPagePreview': True,
                'protectContent': True
            }
        }
    }
)

data = response.json()
print(data)
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

### Option Details

#### Disable Link Preview (`disableWebPagePreview`)

When enabled, Telegram will not generate preview cards for URLs in your message. Useful for:
- Keeping messages compact
- Avoiding preview images that may not represent your content well
- Reducing visual clutter in channels with frequent link sharing

#### Silent Notification (`disableNotification`)

When enabled, recipients will receive the message without a notification sound. The message still appears in their chat list and shows an unread badge. Useful for:
- Posting during off-hours without disturbing subscribers
- High-frequency update channels
- Non-urgent announcements

#### Protect Content (`protectContent`)

When enabled, Telegram restricts what recipients can do with your message:
- Cannot forward the message to other chats
- Cannot save media to device
- Cannot copy text (on mobile apps)
- Screenshot restrictions on some platforms

Useful for:
- Exclusive or premium content
- Time-sensitive announcements
- Content you want to keep within your channel

> **Note:** Content protection is enforced by Telegram clients but is not 100% foolproof. Users with modified clients or screen capture tools may still be able to save content.

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
    content: '*Product Update v2.5*\n\nWe have shipped the following improvements:\n\n- _Faster API response times_ (avg 45ms)\n- New `batch` endpoint for bulk operations\n- Improved error messages with `error_code` field\n\n> This update is backward compatible. No migration needed.\n\nFull changelog: [docs.example.com/changelog](https://docs.example.com/changelog)',
    platforms: ['telegram--1001234567890']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Python (requests)**

```python
import requests

content = """*Product Update v2.5*

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
        'platforms': ['telegram--1001234567890']
    }
)

data = response.json()
print(data)
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "*Product Update v2.5*\n\nWe have shipped the following improvements:\n\n- _Faster API response times_ (avg 45ms)\n- New `batch` endpoint for bulk operations\n- Improved error messages with `error_code` field\n\n> This update is backward compatible. No migration needed.\n\nFull changelog: [docs.example.com/changelog](https://docs.example.com/changelog)",
    "platforms": ["telegram--1001234567890"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const content = `*Product Update v2.5*

We have shipped the following improvements:

- _Faster API response times_ (avg 45ms)
- New \`batch\` endpoint for bulk operations
- Improved error messages with \`error_code\` field

> This update is backward compatible. No migration needed.

Full changelog: [docs.example.com/changelog](https://docs.example.com/changelog)`;

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content,
  platforms: ['telegram--1001234567890']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
    content: '*New Dashboard Preview*\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.',
    platforms: ['telegram--1001234567890']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
        'content': '*New Dashboard Preview*\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.',
        'platforms': ['telegram--1001234567890']
    }
)

data = response.json()
print(data)
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "*New Dashboard Preview*\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.",
    "platforms": ["telegram--1001234567890"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: '*New Dashboard Preview*\n\nHere is a sneak peek at our redesigned analytics dashboard. Key improvements include real-time data refresh and customizable widgets.',
  platforms: ['telegram--1001234567890']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

> **Note:** To attach media to a Telegram post, first create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

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
    content: '*Feature Demo*\n\nWatch our 60-second demo of the new collaboration tools.',
    platforms: ['telegram--1001234567890']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
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
        'content': '*Feature Demo*\n\nWatch our 60-second demo of the new collaboration tools.',
        'platforms': ['telegram--1001234567890']
    }
)

data = response.json()
print(data)
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "*Feature Demo*\n\nWatch our 60-second demo of the new collaboration tools.",
    "platforms": ["telegram--1001234567890"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: '*Feature Demo*\n\nWatch our 60-second demo of the new collaboration tools.',
  platforms: ['telegram--1001234567890']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

## Platform Quirks

- **Bot must be admin**: The Telegram bot must be added as an administrator to the target channel or supergroup, with the `can_post_messages` permission in a channel. The admin permission check happens at **connection time** and again when you call [`test-connection`](../endpoints/test-connection.md#telegram-connections), not at posting time — if the bot lacks these rights, the connection will be rejected during setup. If the bot loses posting rights later, `test-connection` returns `Bot needs channel administrator posting rights`, or `Request failed with status code <code>` if the bot was removed from a private channel.
- **Caption overflow behavior**: When posting with media, the text is sent as a caption (1,024-character limit via bot API). If the content exceeds 1,024 characters and media is attached, the text is sent as a separate reply message to the media post rather than being truncated or rejected. Text-only messages support up to 4,096 characters via MTProto.
- **Caption validation gap**: Publora's validation layer (`postValidationService.js`) does **not** currently validate caption length for Telegram. A 1,024-character caption limit exists at the Telegram API level, but Publora does not pre-check this. If your caption exceeds 1,024 characters on a media post, the error will come from Telegram's API at publish time, not from Publora's validation.
- **WebP auto-conversion**: WebP images are automatically converted to JPEG before sending to Telegram.
- **Markdown formatting**: Telegram supports a subset of markdown. Publora's `safeParseMarkdown` uses single asterisks for bold: `*bold*`, `_italic_`, `` `code` ``, `[text](url)`, and `> blockquote`. Note that this differs from standard Markdown where `**double asterisks**` denote bold. Not all markdown features are supported (e.g., headers are not rendered as headers).
- **Mixed media types not supported**: A single post cannot contain both images and videos. Attempting to send mixed media types will result in an error. Note that Publora's backend validation does **not** enforce mixed media type restrictions for Telegram (it only does so for Instagram). If Telegram rejects mixed media, the error comes from the Telegram API itself, not from Publora's validation layer.
- **Bot connection media group limitations**: Bot connections support either a single video OR multiple photos in a media group, but not multiple videos. Multi-video posts are not supported for bot connections. If you need to post multiple videos, create separate posts for each video.
- **Channel identifier format**: For a public channel, provide its username with or without the `@` prefix, or its `t.me/mychannel` link. For a private channel without a public `@username`, provide its `-100…` chat ID or a `t.me/c/…` link to any of its messages. Invite links (`t.me/+…`, `t.me/joinchat/…`) and public message links (`t.me/mychannel/123`) are not accepted.
- **Private channels have no username**: A channel without a public username is saved with `username: null`; its title is in `displayName`. Label such connections by `displayName`, and keep targeting posts by `platformId`.
- **Bot token security**: Your bot token is stored by Publora in the connection record and is never exposed in API responses. Treat your bot token like a password. Bot tokens are stored in plaintext in the connection record. Encryption at rest is planned but not yet implemented. MTProto sessions, however, ARE encrypted at rest (the service user's session is stored encrypted and decrypted at runtime).
- **MTProto test-connection limitation**: The `test-connection` endpoint always fails for MTProto connections. The Telegram validator only checks bot connections, so an MTProto connection (`connectionType: 'mtproto'`) returns `status: "error"` with the message `Invalid bot connection`. This is a known limitation; MTProto connections work correctly for posting despite failing the test-connection check.
- **Message editing**: Once posted, Telegram messages can be edited, but this capability is not currently exposed through the Publora API.

## API Limits

Understanding the difference between Bot API limits and regular user limits is critical for successful Telegram integration.

### Character Limits

| Element | Limit |
|---------|-------|
| Text message | 4,096 characters |
| **Bot caption** | **1,024 characters** (bots cannot get Premium 2,048 caption limit; the 4,096 limit is for text-only messages, not captions) |
| Bot username | 32 characters |
| Channel name | 128 characters |

### Image Limits (Bot API)

| Property | Limit |
|----------|-------|
| Max size | 10 MB |
| Max count | 10 images per media group |
| Formats | JPEG, PNG, GIF, WebP, BMP |

### Video Limits (Bot API)

| Property | Limit |
|----------|-------|
| Duration | No limit (only size matters) |
| **Max size** | **50 MB** (NOT 4 GB - that's for regular users!) |
| Local Bot API Server | 2 GB max |
| Formats | MP4, MOV, AVI, MKV, WebM |

### CRITICAL: Bot API vs User Limits

Bots have significantly lower limits than regular Telegram users. This is a common source of errors:

| Limit | Bot API | Regular Users |
|-------|---------|---------------|
| File size | 50 MB | 4 GB |
| Caption | 1,024 chars | 4,096 chars |

### Common Error Messages

| Error | Cause |
|-------|-------|
| `MEDIA_CAPTION_TOO_LONG` | Caption exceeds 1,024 characters (bots) |
| `Bad Request: file is too big` | File exceeds 50 MB |

### Rate Limits

Telegram-side message quotas are external, context-dependent, and not a Publora numeric contract. Consult Telegram's current Bot API documentation.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
