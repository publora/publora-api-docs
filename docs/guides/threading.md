# Threading Guide - Post Multi-Part Threads via API

Learn how to post connected threads to Twitter/X and Meta Threads. Both platforms accept one `content` field and publish it as a chain of connected posts.

> **Threads needs one permission.** A Threads chain requires `threads_manage_replies` on the connection. Publora checks the grant when you schedule (from a per-connection cache up to five minutes old) and re-checks it against Meta immediately before publishing, so a missing permission fails the post outright rather than half-publishing it. Reconnect the account in Publora **Channels** to grant it. Single Threads posts are unaffected.

## What is a Thread?

A thread is a series of connected posts that appear as a single conversation. On X/Twitter, these are called "tweet threads" or "tweetstorms." On Threads (by Meta), they appear as connected replies to your own posts. Publora publishes both from a single `content` field.

### Keywords: post thread API, Twitter thread API, Threads multi-post API, tweet thread programmatically, create thread API, multi-tweet API, thread posting automation, social media thread API

## Quick Start

Post a thread by simply providing content that exceeds the platform's character limit:

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `This is a long post that will be automatically split into multiple parts.

When X/Twitter content exceeds the applicable account limit, Publora can automatically create a thread.

Each part is posted as a reply to the previous one, creating a connected conversation that your followers can read through.`,
    platforms: ['twitter-123']
  })
});
```

Publora will:
1. Detect content exceeds the limit
2. Split at natural break points (paragraphs, sentences)
3. Post each part as a reply to the previous
4. Return the head post ID

## How Threading Works

### Automatic Thread Creation

When your content exceeds platform limits, Publora automatically:

1. **Splits content intelligently:**
   - First tries paragraph breaks (`\n\n`)
   - Falls back to sentence endings (`. `, `! `, `? `)
   - Falls back to word boundaries
   - Hard splits only as last resort

2. **Posts sequentially:**
   - First post published normally
   - Each subsequent post replies to the previous
   - Uses `in_reply_to_tweet_id` on X and `reply_to_id` on Threads

3. **Adds numbering to automatic splits:**
   - Appends ` (1/N)`, ` (2/N)`, etc. to each part, on X and on Threads
   - Reserves 10 characters of the per-part budget for the marker
   - Applies **only** when Publora did the splitting. Parts you separate yourself with `---` or `[n/m]` are published exactly as written

### Platform Limits

| Platform | Character Limit | Thread Numbering |
|----------|----------------|------------------|
| X/Twitter (Standard) | 280 | Yes `(1/N)` on automatic splits |
| X/Twitter (Premium) | 25,000 | Yes `(1/N)` on automatic splits |
| Threads | 500 per part | Yes `(1/N)` on automatic splits |

Emoji and other astral characters count as **2** toward these limits.

## Manual Thread Control

### Method 1: Triple Dash Separator

Use `---` to explicitly define where thread breaks should occur:

```javascript
const content = `First part of my thread - the hook that grabs attention.

---

Second part with the main content and details.

---

Final part with the call to action!`;

const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content,
    platforms: ['twitter-123']
  })
});
```

### Method 2: Explicit Markers

Use `[n/m]` markers to define parts:

```javascript
const content = `The hook that grabs attention [1/3]

The main content with all the details you want to share [2/3]

The conclusion with a call to action! [3/3]`;

const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content,
    platforms: ['twitter-123']
  })
});
```

When explicit markers are detected:
- Publora preserves them exactly as written
- Content is split at marker positions
- No additional numbering is added

> **Note:** The `[n/m]` explicit marker and `---` separator behavior is handled by the content splitting service and cannot be independently verified from the core backend source. Thread numbering reserves approximately 10 characters for the `(n/m)` marker when calculating available space per part.

## Media in Threads

Media (images, videos) are attached to the **first post only**. Use the standard upload workflow: create draft, upload media, then schedule.

```javascript
const API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

// Step 1: Create draft post (no scheduledTime)
const postResponse = await fetch(`${BASE_URL}/create-post`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    content: `Check out these results from our latest experiment!

---

The methodology was straightforward but the results surprised us.

---

What do you think? Drop your thoughts below!`,
    platforms: ['twitter-123']
    // No scheduledTime = draft
  })
});

const { postGroupId } = await postResponse.json();

// Step 2: Get pre-signed upload URL
const urlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    fileName: 'experiment-results.jpg',
    contentType: 'image/jpeg',
    type: 'image',
    postGroupId
  })
});
const { uploadUrl, fileUrl } = await urlResponse.json();

// Step 3: Upload file to S3
const fileBuffer = await fs.promises.readFile('./experiment-results.jpg');
await fetch(uploadUrl, {
  method: 'PUT',
  headers: { 'Content-Type': 'image/jpeg' },
  body: fileBuffer
});
console.log(`Uploaded: ${fileUrl}`);

// Step 4: Schedule the thread
await fetch(`${BASE_URL}/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: '2026-03-01T14:00:00.000Z'
  })
});
```

### Media Limits per Platform

| Platform | Images | Video | Notes |
|----------|--------|-------|-------|
| X/Twitter | Up to 4 | 1 | Cannot mix images and video |
| Threads | Up to 20 images | 1 | WebP auto-converted. A chain's first part keeps each carousel item's own type, so a mixed image/video carousel is possible there |

## Cross-Platform Threading

Post the same thread to multiple platforms simultaneously:

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `A long piece of content that will become a thread...`,
    platforms: ['twitter-123']
  })
});
```

Publora handles platform differences:
- X/Twitter: 280 char limit, adds `(1/N)` markers to automatic splits
- Threads: 500 characters per part, same `(1/N)` markers on automatic splits, and the chain needs `threads_manage_replies`

## Scheduling Threads

Schedule a thread for future publication:

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `Your long-form thread content here...`,
    platforms: ['twitter-123'],
    scheduledTime: '2026-03-15T14:30:00.000Z'
  })
});
```

## Error Handling

### Partial Thread Failures

Publora stores one `ScheduledPost` per platform target, not one row per thread part — but `get-post` does report per-part progress in `threadParts[]`. If a platform publishes some parts and then fails, the target is marked failed with `THREAD_PARTIALLY_PUBLISHED`, `postedId` preserves the first part's ID, and each part carries its own `status` and `publishedId`. A group containing only that target is therefore `failed`; a multi-target group can be `partially_published` when another target succeeds.

```json
{
  "success": true,
  "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d",
  "status": "failed",
  "scheduledTime": "2026-03-15T14:30:00.000Z",
  "platformSettings": {},
  "platforms": ["twitter-123"],
  "posts": [
    {
      "platform": "twitter",
      "platformId": "123",
      "content": "Original long thread content",
      "status": "failed",
      "postedId": "1234567890",
      "permalink": null,
      "isThread": true,
      "threadParts": [
        { "index": 0, "content": "Part one (1/3)", "status": "published", "publishedId": "1234567890" },
        { "index": 1, "content": "Part two (2/3)", "status": "published", "publishedId": "1234567891" },
        { "index": 2, "content": "Part three (3/3)", "status": "failed", "publishedId": null }
      ],
      "error": { "code": "THREAD_PARTIALLY_PUBLISHED", "message": "Twitter thread partially published (2/3): Rate limit exceeded", "failedAt": "2026-03-15T14:01:12.000Z", "retryable": false }
    }
  ],
  "media": []
}
```

Already-published parts **are** live. Read `threadParts[]` to see exactly which ones: an entry with `status: "published"` and a `publishedId` exists on the platform. A partially published chain is terminal and non-retryable, so Publora does not re-run it; on Threads a re-entry guard additionally refuses any chain that already has published parts. Publish only the missing parts yourself — recreating the whole post duplicates the ones that already landed.

A third outcome is `PUBLISH_OUTCOME_UNKNOWN`: the platform accepted a publish request but returned no ID, so Publora cannot tell whether that part went out. Automatic retry is disabled for this case by design. Check the live account before sending anything else.

### Rate Limits

Platform-side quotas and pricing change independently and are not a Publora numeric contract. Publora imposes no cap on the number of parts in a chain.

## Best Practices

1. **Keep threads focused** - Each thread should have one main topic
2. **Hook in first post** - The first post should grab attention
3. **Natural breaks** - Use paragraphs to help automatic splitting
4. **Preview before posting** - Long threads are hard to edit once live
5. **Consider your audience** - Some followers prefer single posts over threads

## API Response

The `create-post` endpoint returns the post group reference and effective schedule time, not individual thread parts:

```json
{
  "success": true,
  "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d",
  "scheduledTime": null
}
```

`GET /api/v1/get-post/:postGroupId` returns one entry per connection target, with the stored original content, the target status, and the per-part breakdown:

```json
{
  "success": true,
  "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d",
  "status": "published",
  "scheduledTime": "2026-03-15T14:30:00.000Z",
  "platformSettings": {},
  "platforms": ["twitter-123"],
  "posts": [
    {
      "platform": "twitter",
      "platformId": "123",
      "content": "Original long thread content",
      "status": "published",
      "postedId": "1234567890",
      "permalink": null,
      "isThread": true,
      "threadParts": [
        { "index": 0, "content": "Part one (1/2)", "status": "published", "publishedId": "1234567890" },
        { "index": 1, "content": "Part two (2/2)", "status": "published", "publishedId": "1234567891" }
      ]
    }
  ],
  "media": []
}
```

`postedId` is the **first** part's platform ID, identical to `threadParts[0].publishedId`; the remaining IDs are in `threadParts[]`. No single ID addresses the whole chain. `list-posts` omits `threadParts`, so fetch the group to read chain progress.

## Related Guides

- [X/Twitter Platform Guide](../platforms/x-twitter.md)
- [Threads Platform Guide](../platforms/threads.md)
- [Scheduling Posts](./scheduling.md)
- [Media Uploads](./media-uploads.md)

---

*[Publora](https://publora.com) - Post threads to X/Twitter and Meta Threads via a simple REST API.*
