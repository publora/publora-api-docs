# Threading Guide - Post Multi-Part Threads via API

Learn how to post connected threads to Twitter/X. Multi-part publishing on Meta Threads is currently disabled; any Threads mechanics shown are non-operational reference material.

> **⚠️ Threads Platform Notice:** Multi-part nested threads on Threads are temporarily unavailable due to API access requirements. Twitter/X threading works normally. Single posts and carousels on Threads continue to work. Contact support@publora.com for updates.

## What is a Thread?

A thread is a series of connected posts that appear as a single conversation. On X/Twitter, these are called "tweet threads" or "tweetstorms." On Threads (by Meta), they appear as connected replies to your own posts.

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
   - Uses `in_reply_to_tweet_id` on X; the analogous Threads flow is disabled

3. **Adds numbering (X/Twitter only):**
   - Appends `(1/N)`, `(2/N)`, etc. to each post
   - Reserves 10 characters for the marker

### Platform Limits

| Platform | Character Limit | Thread Numbering |
|----------|----------------|------------------|
| X/Twitter (Standard) | 280 | Yes `(1/N)` |
| X/Twitter (Premium) | 25,000 | Yes `(1/N)` |
| Threads | 500 | Disabled; numbering is not a public contract |

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
| Threads | Up to 20 images | 1 | WebP auto-converted; threading disabled |

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
- X/Twitter: 280 char limit, adds `(1/N)` markers
- Threads: multi-part threading is disabled; numbering semantics are not a public contract until re-enabled

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

If a thread partially publishes (some posts succeed, others fail):

```json
{
  "status": "partially_published",
  "posts": [
    { "platform": "twitter", "platformId": "123", "content": "Part 1 (1/5)", "status": "published", "postedId": "123" },
    { "platform": "twitter", "platformId": "123", "content": "Part 2 (2/5)", "status": "published", "postedId": "456" },
    { "platform": "twitter", "platformId": "123", "content": "Part 3 (3/5)", "status": "failed", "error": { "code": "RATE_LIMIT", "message": "Rate limit exceeded", "platformStatusCode": 429, "platformError": null, "failedAt": "2026-03-15T14:01:12.000Z", "retryable": true } },
    { "platform": "twitter", "platformId": "123", "content": "Part 4 (4/5)", "status": "failed", "error": { "code": "RATE_LIMIT", "message": "Rate limit exceeded", "platformStatusCode": 429, "platformError": null, "failedAt": "2026-03-15T14:01:12.000Z", "retryable": true } },
    { "platform": "twitter", "platformId": "123", "content": "Part 5 (5/5)", "status": "failed", "error": { "code": "RATE_LIMIT", "message": "Rate limit exceeded", "platformStatusCode": 429, "platformError": null, "failedAt": "2026-03-15T14:01:12.000Z", "retryable": true } }
  ]
}
```

The successfully posted parts remain live. You may need to:
1. Wait for rate limits to reset
2. Manually post remaining content as replies

### Rate Limits

Platform-side quotas and pricing change independently and are not a Publora numeric contract. Threads multi-part publishing remains disabled.

## Best Practices

1. **Keep threads focused** - Each thread should have one main topic
2. **Hook in first post** - The first post should grab attention
3. **Natural breaks** - Use paragraphs to help automatic splitting
4. **Preview before posting** - Long threads are hard to edit once live
5. **Consider your audience** - Some followers prefer single posts over threads

## API Response

The `create-post` endpoint returns only the post group reference, not the individual thread parts:

```json
{
  "success": true,
  "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d"
}
```

To see the individual thread parts and their statuses, fetch the post group after it has been processed using `GET /api/v1/get-post/:postGroupId`. Each thread part appears as a separate entry in the `posts` array:

```json
{
  "success": true,
  "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d",
  "posts": [
    {
      "platform": "twitter",
      "platformId": "123",
      "content": "First part (1/3)",
      "status": "published",
      "postedId": "1234567890"
    },
    {
      "platform": "twitter",
      "platformId": "123",
      "content": "Second part (2/3)",
      "status": "published",
      "postedId": "1234567891"
    },
    {
      "platform": "twitter",
      "platformId": "123",
      "content": "Third part (3/3)",
      "status": "published",
      "postedId": "1234567892"
    }
  ]
}
```

The `postedId` field contains the platform-native post ID (not a full URL). If a thread partially publishes, some entries will have `status: "failed"` with an `error` field.

## Related Guides

- [X/Twitter Platform Guide](../platforms/x-twitter.md)
- [Threads Platform Guide](../platforms/threads.md)
- [Scheduling Posts](./scheduling.md)
- [Media Uploads](./media-uploads.md)

---

*[Publora](https://publora.com) - Post threads to X/Twitter via a simple REST API; Meta Threads multi-part publishing is currently disabled.*
