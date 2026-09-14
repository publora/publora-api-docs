# Threads Multi-Post Workflow

Publish a connected chain of Meta Threads posts from one API call. Publora creates each container, publishes it, and links it as a reply to the previous part.

> **One permission is required.** The connection must have granted `threads_manage_replies`. Publora checks the grant when you schedule a chain — from a per-connection cache up to five minutes old — and re-checks it against Meta immediately before publishing, so a missing permission is rejected with `THREADS_PERMISSION_REQUIRED` **before anything is published** — reconnect that account in Publora **Channels** and approve all requested permissions. Single Threads posts never trigger this check.

## Threads Multi-Post API Overview

On Threads (by Meta), you can create connected posts that appear as a conversation thread. This requires:
1. Creating a media container for the first post
2. Publishing it
3. Creating subsequent posts with `reply_to_id` pointing to the previous post
4. Publishing each in sequence

Publora does all four steps for you from a single `create-post` call.

### Keywords: Threads API, Threads multi-post API, Meta Threads API, post to Threads API, Threads thread API, Threads reply chain API, Threads automation API, create Threads post programmatically, Threads bot API, Instagram Threads API, publish multiple Threads posts

## Quick Example

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `I spent 3 months rebuilding our entire backend. Here's what I learned about technical debt.

When we started, our codebase had grown organically over 4 years. What began as a simple MVP had become a tangled mess of shortcuts, deprecated patterns, and "temporary" fixes that became permanent.

The first step was painful: mapping every dependency. We discovered circular imports, dead code paths, and services that nobody remembered building. This audit alone took 3 weeks.

The rebuild itself taught us that incremental migration beats big-bang rewrites. We ran old and new systems in parallel, comparing outputs in real-time. When something broke, we could instantly rollback.

Key lesson: Technical debt isn't just slow code. It's the cognitive load on every engineer who touches your codebase. The rebuild cut our onboarding time from 2 weeks to 3 days.`,
    platforms: ['threads-789']
  })
});
```

Content that exceeds 500 characters is split into a chain automatically, and each automatically derived part gets a ` (1/N)` suffix. Ten characters of every part's 500 are reserved for that suffix, so about 490 characters of your own text fit per part. Emoji count as 2.

## How Threads Threading Works

### The Official Threads API Way

Using Meta's Graph API directly requires multiple steps:

```javascript
// Step 1: Create first post container
const container1 = await fetch(
  `https://graph.threads.net/me/threads?media_type=TEXT&text=First+post&access_token=${token}`,
  { method: 'POST' }
);
const { id: containerId1 } = await container1.json();

// Step 2: Publish first post
const post1 = await fetch(
  `https://graph.threads.net/me/threads_publish?creation_id=${containerId1}&access_token=${token}`,
  { method: 'POST' }
);
const { id: postId1 } = await post1.json();

// Step 3: Create second post with reply_to_id
const container2 = await fetch(
  `https://graph.threads.net/me/threads?media_type=TEXT&text=Second+post&reply_to_id=${postId1}&access_token=${token}`,
  { method: 'POST' }
);
// ... and so on
```

### The Publora way

```javascript
// Single request - everything handled automatically
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: "First post\n\n---\n\nSecond post\n\n---\n\nThird post",
    platforms: ['threads-789']
  })
});
```

## Thread Formatting Options

### Automatic Splitting

Content over 500 characters is split for you. Publora prefers paragraph breaks, then line breaks, then sentence endings, then word boundaries, and cuts mid-word only when nothing better is available. Automatically derived parts are numbered ` (1/N)`.

### Manual Thread Parts with `---`

A `---` break must sit on its own line with a newline before and after it. A `---` at the very start or very end of `content`, or `----`, is not recognised. Parts you separate this way are published exactly as written — **no numbering is added**. If one of them still exceeds 500 characters, Publora splits that part further (again without numbering).

```javascript
const content = `Hot take: Most "productivity" advice is just procrastination in disguise.

---

We spend hours optimizing our Notion setup instead of doing the actual work. We read about how successful people structure their days instead of structuring our own.

---

The most productive people I know have embarrassingly simple systems. A paper notebook. A basic to-do list. Maybe a calendar.

---

The tool doesn't matter. The work does. Stop optimizing, start shipping.`;
```

### Manual Thread Parts with Markers

`[n/m]` markers also split the post, and are kept verbatim. They must form a complete, consistent set — `[1/4]` through `[4/4]`, all sharing the same total; an incomplete or mismatched set is ignored and the text is treated as ordinary content. Marker parts are **not** re-split, so a marker part over 500 characters is rejected when you schedule, with `400` and `THREAD_PART_TOO_LONG`.

The numbers below are written into the text itself rather than as `[n/m]` markers, so they are simply part of the content:

```javascript
const content = `Hot take: Most "productivity" advice is procrastination in disguise. (1/4)

We spend hours optimizing systems instead of doing actual work. (2/4)

The most productive people have embarrassingly simple systems. (3/4)

The tool doesn't matter. The work does. Start shipping. (4/4)`;
```

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 500 characters |
| Hashtags | 1 per post (platform limit) |

## Adding Media to Threads

Media attaches to the **first post** only:

```javascript
// Step 1: Create thread
const postResponse = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `Just launched our new product! Here's a quick walkthrough.

---

The main feature is the dashboard. It shows all your metrics in real-time with zero configuration.

---

We're offering 50% off for early adopters. Link in bio!`,
    platforms: ['threads-789']
  })
});

const { postGroupId } = await postResponse.json();

// Step 2: Get upload URL for image
const urlRes = await fetch('https://api.publora.com/api/v1/get-upload-url', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    fileName: 'screenshot.jpg',
    contentType: 'image/jpeg',
    type: 'image',
    postGroupId
  })
});
const { uploadUrl } = await urlRes.json();

// Step 3: Upload file directly to S3
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'Content-Type': 'image/jpeg' },
  body: screenshotBuffer
});
```

### Media Options

| Type | Limit | Notes |
|------|-------|-------|
| Images | Up to 10 (carousel) | WebP auto-converted to JPEG |
| Video | 1 | MP4 or MOV format |
| Carousel | Up to 10 items | Images only |

## Scheduling Threads Posts

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `Your long thread content...`,
    platforms: ['threads-789'],
    scheduledTime: '2026-03-15T09:00:00.000Z'
  })
});
```

## Rate Limits

Threads-side quotas are advisory, account-dependent, and not a Publora numeric contract. Publora imposes no cap on the number of parts in a chain; parts are published sequentially with a short pause between them.

## Error Handling

### Partial Thread Failure

If a part fails after earlier parts published, the target ends `failed` with `THREAD_PARTIALLY_PUBLISHED`. **Publora does not republish a chain from the beginning** — that would duplicate the messages already live.

`GET /get-post/:postGroupId` shows exactly where it stopped: the target carries `isThread: true` and one `threadParts[]` entry per part, each with `index` (zero-based), `content`, `status` (`pending`, `published`, `failed`) and `publishedId` (`null` until confirmed). The target's `postedId` is the first part's ID.

```json
{
  "platform": "threads",
  "platformId": "17841412345678",
  "status": "failed",
  "postedId": "17900000000000001",
  "isThread": true,
  "threadParts": [
    { "index": 0, "content": "First message.", "status": "published", "publishedId": "17900000000000001" },
    { "index": 1, "content": "Second message.", "status": "failed", "publishedId": null }
  ],
  "error": { "code": "THREAD_PARTIALLY_PUBLISHED", "message": "Threads thread partially published (1/2): …", "retryable": false }
}
```

Read the parts, check the live account, then publish only what is missing. A separate outcome, `PUBLISH_OUTCOME_UNKNOWN`, means Threads accepted a publish request without returning an ID — the outcome is genuinely unknown and automatic retry is deliberately disabled.

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| `THREADS_PERMISSION_REQUIRED` | Connection has not granted `threads_manage_replies` | Reconnect the account in Channels; nothing was published |
| `THREADS_PERMISSIONS_UNAVAILABLE` (503) | Publora's permission lookup failed — not an account problem | Retry scheduling shortly; do not ask the user to reconnect |
| `THREADS_INVALID_THREAD_PART` | A part is empty, not text, or over 500 characters including numbering | Fix the content and create a new post |
| `THREAD_PARTIALLY_PUBLISHED` | A part failed after earlier parts published | Read `threadParts`, publish only the missing parts |
| Rate limit | Too many posts | Wait and retry |
| Media processing | Video still encoding | Retry after delay |
| Invalid media | Unsupported format | Use JPEG/PNG/MP4 |

## Python Example

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': '''Unpopular opinion: You don't need a personal brand.

---

What you need is to be genuinely helpful and share what you're learning. That's it.

---

The "build your personal brand" industry has convinced us that we need logos, color palettes, and "content pillars."

---

But the creators I actually follow? They just share interesting stuff consistently. No strategy deck required.

---

Be useful. Be consistent. Be yourself. That's the whole playbook.''',
        'platforms': ['threads-789']
    }
)

print(response.json())
```

## cURL Example

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "First post of my thread\n\n---\n\nSecond post continues the thought\n\n---\n\nFinal post wraps it up!",
    "platforms": ["threads-789"]
  }'
```

## Why Use Publora for Threads?

| Feature | Publora | Threads API Direct |
|---------|---------|-------------------|
| Thread creation | One API call | Multiple sequential calls |
| OAuth handling | Not required | Complex Meta OAuth flow |
| Content splitting | Automatic, with numbering | Manual implementation |
| API access | Instant | Requires Meta app review |
| Multi-platform | Yes (+ Twitter, LinkedIn, etc.) | Threads only |

## Platform Quirks

- **Single hashtag limit:** Threads allows maximum 1 hashtag per post
- **WebP auto-conversion:** Publora converts WebP images automatically
- **No edit support:** Posted content cannot be edited via API
- **Video processing:** Videos may take time to process before publishing
- **Media on the first part only:** replies in a chain are text-only
- **Reply control applies to the first part only:** `platformSettings.threads.replyControl` is sent with the head post
- **No public `parts` input:** `content` with `---` or `[n/m]` is the whole interface; there is no numbering or threading switch in REST or MCP

## Related

- [Threading Guide](./threading.md) - Complete threading documentation
- [Threads Platform Reference](../platforms/threads.md) - Full platform details
- [Scheduling Posts](./scheduling.md) - Schedule threads for later
- [Media Uploads](./media-uploads.md) - Add images and carousels

---

*[Publora](https://publora.com) — publish a Meta Threads chain from a single REST or MCP call.*
