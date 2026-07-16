# Threads API - Post to Threads via REST API

Post to Threads (by Meta) programmatically using the Publora REST API. A simpler alternative to the official Threads API or Meta Graph API for Threads.

> **⚠️ Temporary Restriction:** Multi-threaded nested posts (content >500 characters that would be split into multiple connected replies) are temporarily unavailable due to Threads app reconnection status. Single posts, carousel posts, and standalone threads continue to work normally. Contact support@publora.com for updates on when this feature will be restored.

## Threads API Overview

Publora provides a unified REST API for publishing single text posts, images, videos, and carousels on Threads. Multi-part thread splitting is currently unavailable.

### Why Use Publora Instead of Threads API / Meta Graph API?

| Feature | Publora API | Threads API (Meta) |
|---------|-------------|-------------------|
| Authentication | Single API key | Meta OAuth 2.0 flow |
| API access | Instant | Requires Meta app review |
| Multi-part thread creation | Disabled | Manual implementation |
| Multi-platform | Post to 10 platforms | Threads only |
| Setup time | 5 minutes | Days to weeks |
| Carousel support | Yes | Yes |

### Keywords: Threads API, Threads posting API, Meta Threads API, post to Threads programmatically, Threads REST API, Threads developer API, Threads automation API, Threads bot API, Instagram Threads API, publish to Threads API

## Platform ID Format

```
threads-{accountId}
```

Where `{accountId}` is your Threads account ID assigned during connection via Meta OAuth.

## Requirements

- A Threads account connected via Meta OAuth through the Publora dashboard
- API key from Publora

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 500 characters |
| Images | Yes | Up to 20 per carousel, WebP auto-converted |
| Videos | Yes | MP4, MOV formats, 1 per post (video carousels not supported by Publora) |
| Carousels | Yes | 2-20 images; video support in carousels is limited (see Platform Quirks) |
| Multi-part threads | Disabled | Numbering and splitting are not a public contract while disabled |
| Hashtags | Yes | Maximum 1 hashtag per post |

## Threading

Multi-part Threads publishing is currently disabled (`supportsThreading: false`). The following mechanics are not a public contract while it remains disabled.

### How It Works

Publora uses the official Threads API `reply_to_id` parameter to chain posts together. Each subsequent post is posted as a reply to the previous one, creating a connected thread visible on Threads.

**Technical flow:**
1. First post is published normally
2. Each subsequent post is published with `reply_to_id` set to the previous post's ID
3. All posts appear as a connected thread on Threads

### Automatic Splitting

Content over 500 characters currently fails validation rather than being split:

Multi-part threading is disabled; splitting and numbering semantics are not a public contract until it is re-enabled.

### Disabled multi-part workflow (reference only)

The following separators are not operational while multi-part Threads publishing is disabled.

**Method 1: Triple dash separator**
```
This is my first post in the thread.

---

This is my second post in the thread.

---

And this is my third post!
```

**Method 2: Explicit markers**
```
First part of the thread [1/3]

Second part of the thread [2/3]

Third and final part [3/3]
```

Explicit marker and numbering behavior is not a public Threads contract while multi-part publishing is disabled.

### Media in Threads

- **Carousel/Images:** Attached to the first post only
- **Video:** Attached to the first post only
- Subsequent posts in the thread are text-only

## Platform-Specific Settings

Threads supports a `replyControl` setting that controls who can reply to your posts:

```json
{
  "platformSettings": {
    "threads": {
      "replyControl": ""
    }
  }
}
```

### Reply Control

| Value | Description |
|-------|-------------|
| `"everyone"` | Anyone can reply. This is a valid enum value in the schema but is distinct from the default `""`. When explicitly set, it is sent to the Threads API. |
| `""` (empty string) | **Default.** No `replyControl` value is sent to the Threads API — the platform's own default behavior applies (anyone can reply). This is NOT included in the API's default platform settings. |
| `"accounts_you_follow"` | Only accounts you follow can reply |
| `"mentioned_only"` | Only accounts mentioned in the post can reply |

## Reply Management

Threads supports reply management through the Publora dashboard. The `getReplies` endpoint retrieves replies to a Threads post, and the `manageReply` endpoint allows you to hide or unhide individual replies. These endpoints are accessible via the dashboard API routes.

> **Note:** Reply management requires the `threads_manage_replies` OAuth scope. The current OAuth flow requests this scope automatically, but older connections established before this scope was added may lack it. If reply management returns permission errors, disconnect and reconnect your Threads account to obtain the updated scopes.

## Examples

### Post a Text Update

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.',
    platforms: ['threads-55667788']
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
        'content': 'Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.',
        'platforms': ['threads-55667788']
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
    "content": "Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.",
    "platforms": ["threads-55667788"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Just shipped a major update to our API. Faster response times, better error messages, and new endpoints for batch operations.',
  platforms: ['threads-55667788']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

### Post with an Image Carousel

Carousels support 2-20 images. The workflow is: create a draft post, upload images, then schedule.

**JavaScript (fetch)**

```javascript
const API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

// Step 1: Create a draft post (no scheduledTime)
const postResponse = await fetch(`${BASE_URL}/create-post`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    content: 'Our product evolution over the past year. #buildinpublic',
    platforms: ['threads-55667788']
    // No scheduledTime = draft
  })
});

const { postGroupId } = await postResponse.json();

// Step 2: Upload each image (2-20 images supported)
const images = ['photo1.jpg', 'photo2.jpg', 'photo3.jpg'];

for (const fileName of images) {
  // Get upload URL
  const uploadUrlResponse = await fetch(`${BASE_URL}/get-upload-url`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': API_KEY
    },
    body: JSON.stringify({
      fileName,
      contentType: 'image/jpeg',
      type: 'image',
      postGroupId
    })
  });

  const { uploadUrl } = await uploadUrlResponse.json();

  // Upload to S3
  const fileBuffer = await fs.promises.readFile(`./${fileName}`);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': 'image/jpeg' },
    body: fileBuffer
  });
}

// Step 3: Schedule the post
await fetch(`${BASE_URL}/update-post/${postGroupId}`, {
  method: 'PUT',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
  },
  body: JSON.stringify({
    status: 'scheduled',
    scheduledTime: '2026-03-15T14:00:00.000Z'
  })
});

console.log('Carousel scheduled!');
```

**Python (requests)**

```python
import requests

API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': API_KEY
}

# Step 1: Create a draft post (no scheduledTime)
post_response = requests.post(
    f'{BASE_URL}/create-post',
    headers=HEADERS,
    json={
        'content': 'Our product evolution over the past year. #buildinpublic',
        'platforms': ['threads-55667788']
        # No scheduledTime = draft
    }
)

post_group_id = post_response.json()['postGroupId']

# Step 2: Upload each image (2-20 images supported)
images = ['photo1.jpg', 'photo2.jpg', 'photo3.jpg']

for file_name in images:
    # Get upload URL
    upload_response = requests.post(
        f'{BASE_URL}/get-upload-url',
        headers=HEADERS,
        json={
            'fileName': file_name,
            'contentType': 'image/jpeg',
            'type': 'image',
            'postGroupId': post_group_id
        }
    )

    upload_url = upload_response.json()['uploadUrl']

    # Upload to S3
    with open(f'./{file_name}', 'rb') as f:
        requests.put(upload_url, headers={'Content-Type': 'image/jpeg'}, data=f.read())

# Step 3: Schedule the post
requests.put(
    f'{BASE_URL}/update-post/{post_group_id}',
    headers=HEADERS,
    json={
        'status': 'scheduled',
        'scheduledTime': '2026-03-15T14:00:00.000Z'
    }
)

print('Carousel scheduled!')
```

**cURL**

```bash
API_KEY="YOUR_API_KEY"

# Step 1: Create a draft post (no scheduledTime)
POST_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d '{
    "content": "Our product evolution over the past year. #buildinpublic",
    "platforms": ["threads-55667788"]
  }')

POST_GROUP_ID=$(echo "$POST_RESPONSE" | jq -r '.postGroupId')

# Step 2: Upload each image (2-20 images supported)
for FILE in photo1.jpg photo2.jpg photo3.jpg; do
  UPLOAD_RESPONSE=$(curl -s -X POST https://api.publora.com/api/v1/get-upload-url \
    -H "Content-Type: application/json" \
    -H "x-publora-key: $API_KEY" \
    -d "{
      \"fileName\": \"$FILE\",
      \"contentType\": \"image/jpeg\",
      \"type\": \"image\",
      \"postGroupId\": \"$POST_GROUP_ID\"
    }")

  UPLOAD_URL=$(echo "$UPLOAD_RESPONSE" | jq -r '.uploadUrl')

  curl -s -X PUT "$UPLOAD_URL" \
    -H "Content-Type: image/jpeg" \
    --data-binary @"./$FILE"
done

# Step 3: Schedule the post
curl -X PUT "https://api.publora.com/api/v1/update-post/$POST_GROUP_ID" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: $API_KEY" \
  -d '{
    "status": "scheduled",
    "scheduledTime": "2026-03-15T14:00:00.000Z"
  }'
```

> **Note:** For immediate publishing, set `scheduledTime` a few seconds in the future — a time already in the past is clamped to server time and flagged with a `SCHEDULED_TIME_COERCED` warning (see [past scheduled times](../guides/scheduling.md#past-scheduled-times)). For more details, see the [media upload workflow](../guides/media-uploads.md).

### Post a Thread (Long Content)

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: `We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.

First, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.

The actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step.`,
    platforms: ['threads-55667788']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Python (requests)**

```python
import requests

content = """We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.

First, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.

The actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step."""

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': content,
        'platforms': ['threads-55667788']
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
    "content": "We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.\n\nFirst, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.\n\nThe actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step.",
    "platforms": ["threads-55667788"]
  }'
# Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const content = `We just completed a major infrastructure migration and I want to share what we learned. Moving from a monolithic architecture to microservices is not as straightforward as the blog posts make it sound.

First, we had to map every single dependency between our services. This alone took two weeks. We discovered circular dependencies we never knew existed and had to refactor several core modules before we could even begin the migration.

The actual migration took three months. We ran both systems in parallel, comparing outputs in real-time. When we finally cut over, we had 99.97% uptime throughout the process. The key was incremental rollout and comprehensive monitoring at every step.`;

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content,
  platforms: ['threads-55667788']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123...", "scheduledTime": null }
```

Multi-part Threads publishing is currently disabled. Long content is not automatically split, and numbering semantics are not a public contract until threading is re-enabled.

## Platform Quirks

- **Single hashtag limit**: Threads allows a maximum of 1 hashtag per post. If your content includes more than one hashtag, only the first will be recognized by the platform.
- **WebP auto-conversion**: If you provide WebP images, Publora automatically converts them to **JPEG** before uploading to Threads.
- **Threading disabled**: Multi-part Threads publishing is disabled; numbering semantics are not a public contract until it is re-enabled.
- **Manual thread parts unavailable**: `---` separators are not operational while multi-part Threads publishing is disabled.
- **No edit support**: Once posted, Threads posts cannot be edited via the API. You would need to delete and repost.
- **MP4 and MOV for videos**: MP4 and MOV video formats are supported. Other formats will be rejected.
- **Carousel video support is limited**: While the Threads API supports videos in carousels, Publora's current implementation only supports IMAGE type items in carousels. Standalone video posts work normally, but VIDEO items within carousels are not yet supported by Publora.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 500 characters |
| Hashtags | 1 per post |
| Carousel items | 2-20 images (video in carousels not yet supported by Publora) |

## API Limits

<!-- limits tables below synced from @publora/platform-limits 1.0.0 (2026-03-11) — regenerate on bump -->

### Text Limits

| Element | Limit |
|---------|-------|
| Post body | 500 characters |
| Links per post | 5 |
| Hashtags | 1 per post |

### Media Limits

| Media Type | Max Size | Max Count | Supported Formats |
|------------|----------|-----------|-------------------|
| Images | 8 MB | 20 per carousel | JPEG, PNG, WebP |
| Videos | 1 GB | 1 per post (video carousels not supported by Publora) | MP4, MOV |

| Video Constraint | Limit |
|------------------|-------|
| Duration | 5 minutes |

### Rate Limits

Threads-side posting quotas are advisory, account-dependent, and may change without notice; they are not a Publora numeric contract.

### Additional Notes

- Multi-part threading is disabled; numbering and splitting semantics are not a public contract until re-enabled.
- Platform-side rate-limit failures are surfaced; do not assume automatic retry or redistribution.

## What you can't do

- **Publish multi-part Threads threads:** Publora currently treats Threads as a single-post target. Manual `---` parts and automatic splitting are disabled because the shared capability flag is `supportsThreading: false`.

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
