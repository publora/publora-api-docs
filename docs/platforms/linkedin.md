# LinkedIn API - Post to LinkedIn via REST API

Post to LinkedIn programmatically using the Publora REST API. A simpler alternative to the LinkedIn Marketing API, LinkedIn Share API, or LinkedIn Community Management API.

## LinkedIn API Overview

Publora provides a unified REST API for professional content publishing to LinkedIn, including text posts, media attachments (images and videos), analytics retrieval (impressions, reactions, comments), and reaction management. No need to manage LinkedIn OAuth flows, handle the LinkedIn API partner program requirements, or implement complex share creation endpoints.

### Why Use Publora Instead of LinkedIn Marketing API?

| Feature | Publora API | LinkedIn Marketing API |
|---------|-------------|------------------------|
| Authentication | Single API key | Complex OAuth 2.0 flow |
| API access | Instant | LinkedIn partner approval |
| Analytics | Built-in | Separate API calls |
| Multi-platform | Post to 10 platforms | LinkedIn only |
| Setup time | 5 minutes | Weeks (partner approval) |
| Reactions | Full support | Full support |

### Keywords: LinkedIn API, LinkedIn posting API, LinkedIn Share API, LinkedIn Marketing API, post to LinkedIn programmatically, LinkedIn REST API, LinkedIn developer API, LinkedIn automation API, LinkedIn content API, LinkedIn bot API, LinkedIn analytics API, LinkedIn UGC API

## Platform ID Format

```
linkedin-{profileId}
```

Where `{profileId}` is your LinkedIn profile identifier assigned during account connection via OAuth.

## Requirements

- A LinkedIn account connected via OAuth through the Publora dashboard
- API key from Publora (Pro or Premium plan required)

> **Note:** The free Starter plan includes API access (`apiAccess: true`) — API key authentication works on all standard plans.

## Supported Content

| Type | Supported | Limits |
|------|-----------|--------|
| Text | Yes | 3,000 characters |
| Images | Yes | JPEG, PNG (WebP auto-converted to JPEG), multiple supported |
| Videos | Yes | MP4 format |
| Analytics | Yes | IMPRESSION, MEMBERS_REACHED, RESHARE, REACTION, COMMENT |
| Reactions | Yes | LIKE, PRAISE, EMPATHY, INTEREST, APPRECIATION, ENTERTAINMENT |
| Comments | Yes | Create, delete, reply (1,250 characters max) |
| Mentions | Yes | @mention people and organizations |

## API Limits

### Character Limit

- **3,000 characters** maximum per post
- First 210 characters visible before "see more" is shown

### Image Limits (API)

| Property | Limit |
|----------|-------|
| Max size | 5 MB |
| Max count | 10 (multi-image posts) |
| Formats | JPEG, PNG, GIF, WebP (WebP images are auto-converted to JPEG before upload to LinkedIn) |

> **Note:** GIF images are passed through without conversion — if the LinkedIn API rejects a GIF, consider converting to JPEG/PNG before uploading.

> **Important:** Organic carousels (swipeable multi-image posts) are NOT supported via the API. The API only supports multi-image grid layouts. Swipeable carousels are only available for sponsored content.

### Video Limits (API)

| Property | Limit |
|----------|-------|
| Duration | 30 minutes (more than native's 15 min desktop / 10 min mobile) |
| Max size | **500 MB** (native allows 5 GB) |
| Formats | MP4 only |

### Important API Restrictions

- **Cannot mix media types**: Images cannot be combined with videos or documents in the same post
- **No organic carousels**: Swipeable multi-image posts are not available via API — only multi-image grid layout
- **Rate limit**: 200+ API calls per hour, based on user count

### Common Error Messages

| Error | Cause |
|-------|-------|
| `MEDIA_ASSET_PROCESSING_FAILED` | File too large or unsupported format |
| `Error 429` | Rate limit exceeded |

## Mentioning People and Organizations

Publora supports @mentioning LinkedIn members and organizations in your posts. When the post is published, the mention becomes a clickable link and the mentioned person/organization receives a notification.

### Mention Syntax

Use the following format in your post content:

```
@{urn:li:person:MEMBER_ID|Display Name}       # Mention a person
@{urn:li:organization:ORG_ID|Company Name}    # Mention an organization
```

### Examples

**Mentioning a person:**
```
Great insights from @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} on building APIs!
```
Result: `Great insights from @Serge Bulaev on building APIs!`

**Mentioning a company:**
```
Excited to work with @{urn:li:organization:107107343|Creative Content Crafts Inc}!
```
Result: `Excited to work with @Creative Content Crafts Inc!`

Both mentions will be rendered as clickable links on LinkedIn.

### How to Find LinkedIn URNs

- **Person URN**: The member ID from LinkedIn's API. The format is `urn:li:person:{member_id}`. Member IDs are typically long alphanumeric strings (e.g. `ACoAABcD1234EfG`). You can find them via LinkedIn's API (`/me` endpoint) or browser developer tools.
- **Organization URN**: Found in the company page URL or via LinkedIn's API. Format: `urn:li:organization:{numeric_id}`. Organization IDs are typically 8-digit numbers.

> **Important:** You must use a valid LinkedIn URN ID. Invalid or made-up IDs will cause a `400` error: `"Person URN ID in commentary field is invalid."` Always verify the URN before posting. See the [LinkedIn Mentions Guide](/docs/guides/linkedin-mentions.md) for details.

### Important: Name Matching Requirements

**The display name must exactly match the name on LinkedIn** (case-sensitive):

- For **people**: Use their exact name as shown on their LinkedIn profile
- For **organizations**: Use the **exact registered company name** including suffixes like "Inc", "LLC", etc.

| Correct | Incorrect |
|---------|-----------|
| `@{urn:li:organization:98765432\|Acme Corp Inc}` | `@{urn:li:organization:98765432\|Acme Corp}` |
| `@{urn:li:person:ACoAADeFgHi5678\|John Smith}` | `@{urn:li:person:ACoAADeFgHi5678\|john smith}` |

If the name doesn't match exactly, the mention will appear as plain text instead of a clickable link.

### Code Example

**JavaScript**
```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Excited to collaborate with @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} on this project!',
    platforms: ['linkedin-987654321']
  })
});
```

**Python**
```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Excited to collaborate with @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} on this project!',
        'platforms': ['linkedin-987654321']
    }
)
```

## Analytics

Publora can retrieve analytics for your LinkedIn posts. Available metrics:

| Metric | Description |
|--------|-------------|
| `IMPRESSION` | Number of times the post was displayed |
| `MEMBERS_REACHED` | Unique LinkedIn members who saw the post |
| `RESHARE` | Number of times the post was shared |
| `REACTION` | Total reactions on the post |
| `COMMENT` | Number of comments on the post |

## Reactions

LinkedIn supports a richer set of reactions than a simple "like":

| Reaction | Description |
|----------|-------------|
| `LIKE` | Standard like (thumbs up) |
| `PRAISE` | Clapping hands / applause |
| `EMPATHY` | Heart / love |
| `INTEREST` | Lightbulb / insightful |
| `APPRECIATION` | Supportive |
| `ENTERTAINMENT` | Funny / laughing |

### Create a Reaction

**Endpoint:** `POST /api/v1/linkedin-reactions`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123` or `urn:li:ugcPost:123`) |
| `reactionType` | string | Yes | One of: `LIKE`, `PRAISE`, `EMPATHY`, `INTEREST`, `APPRECIATION`, `ENTERTAINMENT` |
| `platformId` | string | Yes | Your LinkedIn platform ID (e.g., `linkedin-ABC123`) |

**JavaScript**
```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reactions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:ugcPost:7429953213384187904',
    reactionType: 'INTEREST',
    platformId: 'linkedin-987654321'
  })
});

const data = await response.json();
// HTTP 201 Created
console.log(data);
// {
//   success: true,
//   reaction: {
//     id: "urn:li:reaction:(urn:li:person:xxx,urn:li:ugcPost:xxx)",
//     reactionType: "INTEREST",
//     ...
//   },
//   urnTranslated: { from: "original-urn", to: "translated-urn" }  // present when URN translation was needed
// }
```

> **Note:** The create reaction endpoint returns HTTP **201** (not 200). The response may include an `urnTranslated` field when the provided URN needed to be translated to a different format for the LinkedIn API.

> **Note:** POSTing a reaction that already exists returns **409** `REACTION_ALREADY_EXISTS`. Reactions are keyed by `(account, post)`, so delete the existing reaction before applying a different type.

**cURL**
```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reactions \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:ugcPost:7429953213384187904",
    "reactionType": "INTEREST",
    "platformId": "linkedin-987654321"
  }'
```

### Delete a Reaction

**Endpoint:** `DELETE /api/v1/linkedin-reactions`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN |
| `platformId` | string | Yes | Your LinkedIn platform ID |

**cURL**
```bash
curl -X DELETE https://api.publora.com/api/v1/linkedin-reactions \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:ugcPost:7429953213384187904",
    "platformId": "linkedin-987654321"
  }'
```

> **Note:** The delete reaction response includes the `reaction` field set to `null` (not the deleted reaction type). It may also include an `urnTranslated` field (`{ from: "original-urn", to: "translated-urn" }`) when URN translation was needed.

**Response example:**
```json
{
  "success": true,
  "reaction": null,
  "urnTranslated": { "from": "original-urn", "to": "translated-urn" }
}
```

## Comments

Publora supports creating and deleting comments on LinkedIn posts programmatically.

### Create a Comment

**Endpoint:** `POST /api/v1/linkedin-comments`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123` or `urn:li:ugcPost:123`) |
| `message` | string | Yes | Comment text (max 1,250 characters) |
| `platformId` | string | Yes | Your LinkedIn platform ID (e.g., `linkedin-ABC123`) |
| `parentComment` | string | No | Comment URN for nested replies |

**JavaScript**
```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7434685316856377344',
    message: 'Great post! Thanks for sharing.',
    platformId: 'linkedin-987654321'
  })
});

const data = await response.json();
// HTTP 201 Created
console.log(data);
// {
//   success: true,
//   comment: {
//     id: "7434695495614312448",
//     commentUrn: "urn:li:comment:(urn:li:activity:xxx,7434695495614312448)",
//     message: "Great post! Thanks for sharing.",
//     ...
//   }
// }
```

> **Note:** The create comment endpoint returns HTTP **201** (not 200).

**cURL**
```bash
curl -X POST https://api.publora.com/api/v1/linkedin-comments \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7434685316856377344",
    "message": "Great post! Thanks for sharing.",
    "platformId": "linkedin-987654321"
  }'
```

### Delete a Comment

**Endpoint:** `DELETE /api/v1/linkedin-comments`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN the comment belongs to |
| `commentId` | string | Yes | Comment URN to delete (also accepts plain numeric IDs, e.g., `7434695495614312448`) |
| `platformId` | string | Yes | Your LinkedIn platform ID |

**JavaScript**
```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'DELETE',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7434685316856377344',
    commentId: 'urn:li:comment:(urn:li:activity:xxx,7434695495614312448)',
    platformId: 'linkedin-987654321'
  })
});

const data = await response.json();
console.log(data);
// { success: true, deleted: "urn:li:comment:(...)" }
```

**cURL**
```bash
curl -X DELETE https://api.publora.com/api/v1/linkedin-comments \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7434685316856377344",
    "commentId": "urn:li:comment:(urn:li:activity:xxx,7434695495614312448)",
    "platformId": "linkedin-987654321"
  }'
```

### Replying to a Comment

To reply to an existing comment (nested comment), include the `parentComment` parameter:

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7434685316856377344',
    message: 'I agree with this point!',
    platformId: 'linkedin-987654321',
    parentComment: 'urn:li:comment:(urn:li:activity:xxx,7434695495614312448)'
  })
});
```

## Reshare

Reshare (repost) an existing LinkedIn post to your own feed, with optional commentary. Works for both personal profile and company-page connections — the reshare is authored as the member or the organization automatically.

### Reshare a Post

**Endpoint:** `POST /api/v1/linkedin-reshare`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | Your LinkedIn platform ID (e.g., `linkedin-ABC123`) |
| `parent` | string | Yes | URN of the post to reshare (`urn:li:share:<id>` or `urn:li:ugcPost:<id>`) |
| `commentary` | string | No | Text added above the reshare (max 3,000 characters) |
| `visibility` | string | No | `PUBLIC` or `CONNECTIONS` (case-insensitive). Defaults to `PUBLIC` |

**JavaScript**
```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reshare', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-987654321',
    parent: 'urn:li:share:7434685316856377344',
    commentary: 'Great post! Sharing with my network.',
    visibility: 'PUBLIC'
  })
});

const data = await response.json();
// HTTP 201 Created
console.log(data.reshare.id); // urn:li:share:...
```

> **Note:** The reshare endpoint returns HTTP **201** (not 200). The new reshare URN is in `reshare.id`.

**cURL**
```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reshare \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platformId": "linkedin-987654321",
    "parent": "urn:li:share:7434685316856377344",
    "commentary": "Great post! Sharing with my network."
  }'
```

See the [LinkedIn Reshare endpoint reference](../endpoints/linkedin-reshare.md) for the full parameter and error details.

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
    content: 'Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.',
    platforms: ['linkedin-987654321']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123..." }
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
        'content': 'Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.',
        'platforms': ['linkedin-987654321']
    }
)

data = response.json()
print(data)
# Response: { "success": true, "postGroupId": "abc123..." }
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.",
    "platforms": ["linkedin-987654321"]
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Excited to announce our Series A funding! We are building the future of social media management for developer teams. More details coming soon.',
  platforms: ['linkedin-987654321']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123..." }
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
    content: 'Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!',
    platforms: ['linkedin-987654321']
  })
});

const data = await response.json();
console.log(data);
// Response: { "success": true, "postGroupId": "abc123..." }
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
        'content': 'Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!',
        'platforms': ['linkedin-987654321']
    }
)

data = response.json()
print(data)
# Response: { "success": true, "postGroupId": "abc123..." }
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!",
    "platforms": ["linkedin-987654321"]
  }'
# Response: { "success": true, "postGroupId": "abc123..." }
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/create-post', {
  content: 'Our team just wrapped up an incredible hackathon weekend. Here are some highlights from the event!',
  platforms: ['linkedin-987654321']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
// Response: { "success": true, "postGroupId": "abc123..." }
```

> **Note:** To attach media to a LinkedIn post, first create the post, then upload media using the [media upload workflow](../guides/media-uploads.md) with the returned `postGroupId`.

### Check Post Analytics

**Endpoint:** `POST /api/v1/linkedin-post-statistics`

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123` or `urn:li:ugcPost:123`) |
| `platformId` | string | Yes | Your LinkedIn platform ID (e.g., `linkedin-ABC123`) |
| `queryType` | string | No | Single metric to retrieve. Valid values: `IMPRESSION`, `MEMBERS_REACHED`, `RESHARE`, `REACTION`, `COMMENT`. `ALL` is only valid when used with the `queryTypes` array parameter, not with the singular `queryType` parameter. At least one of `queryType` or `queryTypes` is required. Without this parameter (and without `queryTypes`), the API returns 400: `queryType or queryTypes is required`. |
| `queryTypes` | array of strings | No | Multiple metrics to retrieve in one request. Same valid values as `queryType`. At least one of `queryType` or `queryTypes` is required. |

> **Response format differs by request type:**
> - **Single metric** (using `queryType`): Returns `{ success: true, count: 123, cached: true/false }`
> - **Multiple metrics** (using `queryTypes`): Returns `{ success: true/false, metrics: { IMPRESSION: 4521, ... }, cached: true/false, cachedMetrics: [...], errors: [...] }`
>
> **Multi-metric response fields:**
> - `cached` (boolean): Whether any metrics were served from cache.
> - `cachedMetrics` (array): Only present when non-empty. Lists which metrics came from cache.
> - `errors` (array): Present when some metrics failed to fetch.
> - `success` is `false` when one or more metrics failed to fetch (i.e., `errors` array is non-empty). The `metrics` object is still populated with `null` for failed metrics.

**JavaScript (fetch) — multiple metrics**

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7434685316856377344',
    platformId: 'linkedin-987654321',
    queryTypes: ['IMPRESSION', 'MEMBERS_REACHED', 'RESHARE', 'REACTION', 'COMMENT']
  })
});

const analytics = await response.json();
console.log(analytics);
// {
//   success: true,
//   metrics: {
//     IMPRESSION: 4521,
//     MEMBERS_REACHED: 3200,
//     RESHARE: 12,
//     REACTION: 89,
//     COMMENT: 15
//   }
// }
```

**JavaScript (fetch) — single metric**

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7434685316856377344',
    platformId: 'linkedin-987654321',
    queryType: 'IMPRESSION'
  })
});

const analytics = await response.json();
console.log(analytics);
// { success: true, count: 4521, cached: false }
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-post-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7434685316856377344',
        'platformId': 'linkedin-987654321',
        'queryTypes': ['IMPRESSION', 'MEMBERS_REACHED', 'RESHARE', 'REACTION', 'COMMENT']
    }
)

analytics = response.json()
print(analytics)
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-post-statistics \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7434685316856377344",
    "platformId": "linkedin-987654321",
    "queryTypes": ["IMPRESSION", "MEMBERS_REACHED", "RESHARE", "REACTION", "COMMENT"]
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const response = await axios.post('https://api.publora.com/api/v1/linkedin-post-statistics', {
  postedId: 'urn:li:share:7434685316856377344',
  platformId: 'linkedin-987654321',
  queryTypes: ['IMPRESSION', 'MEMBERS_REACHED', 'RESHARE', 'REACTION', 'COMMENT']
}, {
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

console.log(response.data);
```

## Additional Analytics Endpoints

Beyond post-level statistics, Publora provides additional LinkedIn analytics endpoints:

- **Followers Statistics** (`POST /api/v1/linkedin-followers`) — Retrieve follower demographics and growth data for your LinkedIn account. See the [LinkedIn Followers](../endpoints/linkedin-followers.md) endpoint docs.
- **Account Statistics** (`POST /api/v1/linkedin-account-statistics`) — Get aggregate account-level statistics including total impressions, engagement, and reach. See the [LinkedIn Statistics](../endpoints/linkedin-statistics.md) endpoint docs.
- **Profile Summary** (`POST /api/v1/linkedin-profile-summary`) — Retrieve your LinkedIn profile summary information. See the [LinkedIn Profile Summary](../endpoints/linkedin-profile-summary.md) endpoint docs.

## Platform Quirks

- **URN formats differ**: LinkedIn URLs use `urn:li:activity:xxx` but the API requires `urn:li:share:xxx` or `urn:li:ugcPost:xxx`. For posts created via Publora, use the `postedId` from the `get-post` endpoint. For external posts, you may need to look up the correct URN format.
- **WebP auto-conversion**: If you provide a WebP image URL, Publora automatically converts it to JPEG before uploading to LinkedIn. No action needed on your part.
- **3,000-character limit**: LinkedIn enforces a strict 3,000-character limit for post text. Publora will return an error if your content exceeds this.
- **Multiple images**: LinkedIn supports posting multiple images at once. They will appear as a multi-image grid layout (not a swipeable carousel — organic carousels are not supported via the API).
- **Rich text not supported via API**: LinkedIn's API does not support bold, italic, or other rich text formatting. Use plain text or Unicode characters for emphasis.
- **Analytics delay**: LinkedIn analytics may take up to 24 hours to fully populate. Querying immediately after posting will return partial data.
- **Hashtags**: LinkedIn hashtags are supported in the content body. They are treated as plain text but become clickable on the platform.
- **Reactions on external posts**: You can only react to posts visible in your LinkedIn network. Ensure your account is connected to the post author.

## Character Limits

| Element | Limit |
|---------|-------|
| Post body | 3,000 characters |
| Comment | 1,250 characters (validated by Publora; returns 400 error: `"message cannot exceed 1250 characters"`) |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
