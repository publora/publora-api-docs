# Post Twitter Threads via API - Tweet Thread Automation

Post tweet threads (tweetstorms) to X/Twitter programmatically using the Publora REST API. A simpler alternative to manually chaining tweets with the Twitter API v2.

## Twitter Thread API Overview

Creating a Twitter thread traditionally requires:
1. Posting the first tweet
2. Capturing the tweet ID
3. Posting each subsequent tweet as a reply using `in_reply_to_tweet_id`
4. Handling errors and partial failures

Publora handles all of this automatically with a single API call.

### Keywords: Twitter thread API, tweet thread API, post tweetstorm API, Twitter API thread, X API thread, multi-tweet API, tweet chain API, create Twitter thread programmatically, automate tweet threads, Twitter thread automation, post multiple tweets API

## Quick Example

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `I've been building in public for 6 months. Here's everything I learned.

First, consistency beats intensity. Posting daily, even small updates, builds more momentum than sporadic big announcements.

Second, share your failures. People connect more with struggles than successes. My most engaging posts were about things that went wrong.

Third, engage genuinely. Reply to comments, ask questions, be curious about others' work. The algorithm rewards conversations.

Finally, don't chase vanity metrics. Focus on connecting with the right people, not the most people.`,
    platforms: ['twitter-123456']
  })
});

const data = await response.json();
console.log('Thread posted:', data);
```

This will automatically:
- Split into 4+ tweets at sentence boundaries
- Add `(1/N)` numbering to each tweet
- Post each as a reply to the previous
- Return the Publora post-group ID; per-part tweet IDs remain internal

## How Twitter Threads Work

### The Official Twitter API v2 Way

With the official Twitter API, creating a thread requires multiple sequential requests:

```javascript
// Step 1: Post first tweet
const tweet1 = await client.v2.tweet({ text: "First tweet (1/3)" });

// Step 2: Post reply to first tweet
const tweet2 = await client.v2.tweet({
  text: "Second tweet (2/3)",
  reply: { in_reply_to_tweet_id: tweet1.data.id }
});

// Step 3: Post reply to second tweet
const tweet3 = await client.v2.tweet({
  text: "Third tweet (3/3)",
  reply: { in_reply_to_tweet_id: tweet2.data.id }
});
```

### The Publora Way

```javascript
// Single request - Publora handles everything
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: "First tweet\n\n---\n\nSecond tweet\n\n---\n\nThird tweet",
    platforms: ['twitter-123456']
  })
});
```

## Thread Formatting Options

### Automatic Splitting

Content over the connected account's applicable limit (280 standard / 25,000 Premium or PremiumPlus) is automatically split:
- At paragraph breaks (`\n\n`) first
- At sentence endings (`. `, `! `, `? `) second
- At word boundaries as fallback
- `(1/N)` markers added automatically

### Manual Thread Parts with `---`

```javascript
const content = `Hook: Why most developers fail at Twitter.

---

Mistake #1: They only share finished work. Share the process, the bugs, the learning moments.

---

Mistake #2: They don't engage. Twitter is social. Reply to others, join conversations.

---

Mistake #3: Inconsistency. Tweet daily, even if it's just a quick update.

---

The fix? Treat it like a dev log, not a highlight reel. Your journey IS the content.`;
```

### Manual Thread Parts with Markers

```javascript
const content = `Hook: Why most developers fail at Twitter. (1/5)

Mistake #1: They only share finished work. (2/5)

Mistake #2: They don't engage with others. (3/5)

Mistake #3: Inconsistency in posting schedule. (4/5)

The fix? Treat Twitter like a dev log, not a highlight reel. (5/5)`;
```

## Character Limits

| Account Type | Per-Tweet Limit | Thread Marker Space |
|-------------|-----------------|---------------------|
| Standard | 280 characters | 10 characters reserved for `(X/N)` |
| X Premium | 25,000 characters | 10 characters reserved for `(X/N)` |

Publora automatically reserves space for thread markers.

### Emoji Counting

Emojis count as 2 characters on X/Twitter. Publora accounts for this:

```javascript
// This tweet is 282 "display" characters but 284 Twitter characters
const content = "Hello! 👋 This is a test tweet with emojis 🚀";
// Publora correctly calculates: 280 + 4 (2 emojis × 2) = 284
```

## Adding Media to Threads

Images and videos attach to the **first tweet** only:

```javascript
// Step 1: Create thread
const postResponse = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `Check out this new feature! 🎉

---

Here's how it works: [technical explanation]

---

Try it out and let me know what you think!`,
    platforms: ['twitter-123']
  })
});

const { postGroupId } = await postResponse.json();

// Step 2: Get upload URL for image (goes to first tweet)
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
  body: screenshotFile
});
```

### Media Limits

- **Images:** Up to 4 per tweet (first tweet only)
- **Video:** 1 per tweet (first tweet only)
- **Format:** All images are auto-converted to PNG (1000px max width) before upload to Twitter
- **Note:** Cannot mix images and video in the same tweet

## Scheduling Twitter Threads

> **Note:** `scheduledTime` is optional in both REST and MCP; omitting it creates a draft.

Schedule a thread for optimal posting time:

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `Your thread content here...`,
    platforms: ['twitter-123'],
    scheduledTime: '2026-03-15T14:00:00.000Z'  // 2 PM UTC
  })
});
```

## Rate Limits

X-side pricing and posting quotas change independently and are not a Publora numeric contract. Consult X's current developer documentation; each tweet in a thread is a separate platform publication.

## Error Handling

### Partial Thread Failure

The public API does not return a per-part result object. It stores one platform target for the X connection. On a partial thread failure, that target is failed with `THREAD_PARTIALLY_PUBLISHED`, and `postedId` contains the head tweet ID; the remaining per-part IDs stay internal. A group with only this X target has group status `failed`.

```json
{
  "platform": "twitter",
  "platformId": "123456",
  "status": "failed",
  "postedId": "1234567890",
  "permalink": null,
  "error": { "code": "THREAD_PARTIALLY_PUBLISHED", "message": "Twitter thread partially published (2/5): Rate limit exceeded", "retryable": false }
}
```

Already-published tweets can remain live, but the public response exposes only the head ID and does not enumerate the remaining IDs or identify individual failed parts.

### Common Errors

| Error | Cause | Solution |
|-------|-------|----------|
| Rate limit exceeded | Too many requests | Wait 15 min or upgrade tier |
| Character limit exceeded | Tweet too long | Check emoji counting, reduce content |
| Duplicate content | Same tweet posted recently | Vary your content |

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
        'content': '''Thread: 5 Python tips I wish I knew earlier.

---

1. Use enumerate() instead of range(len()). Cleaner and more Pythonic.

---

2. F-strings are faster than .format() and way more readable.

---

3. List comprehensions aren't just shorter—they're actually faster.

---

4. Use `if __name__ == "__main__":` in every script. Future you will thank you.

---

5. The walrus operator `:=` is underrated. Assign and use in one line.''',
        'platforms': ['twitter-123']
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
    "content": "First tweet of my thread\n\n---\n\nSecond tweet continues the story\n\n---\n\nFinal tweet with the conclusion!",
    "platforms": ["twitter-123456"]
  }'
```

## Why Use Publora for Twitter Threads?

| Feature | Publora | Twitter API v2 Direct |
|---------|---------|----------------------|
| Thread creation | Single API call | Multiple sequential calls |
| Content splitting | Automatic | Manual implementation |
| Error recovery | Built-in partial save | Manual handling |
| Rate limit handling | Automatic errors | Manual tracking |
| Multi-platform | Yes (+ Threads, LinkedIn, etc.) | Twitter only |

## Related

- [Threading Guide](./threading.md) - Complete threading documentation
- [X/Twitter Platform Reference](../platforms/x-twitter.md) - Full platform details
- [Scheduling Posts](./scheduling.md) - Schedule threads for later
- [Media Uploads](./media-uploads.md) - Add images to threads

---

*Post Twitter threads with a single API call using [Publora](https://publora.com).*
