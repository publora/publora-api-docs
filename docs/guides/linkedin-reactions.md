# How to Like and React to LinkedIn Posts with Publora

Add reactions (like, insightful, love, etc.) to LinkedIn posts programmatically using the Publora API.

## Overview

LinkedIn supports 6 different reaction types. Publora allows you to add or remove any of these reactions on posts visible to your connected account.

## Available Reaction Types

| Reaction | Description | Icon |
|----------|-------------|------|
| `LIKE` | Standard like | Thumbs up |
| `PRAISE` | Applause / celebration | Clapping hands |
| `EMPATHY` | Love / support | Heart |
| `INTEREST` | Insightful / thought-provoking | Lightbulb |
| `APPRECIATION` | Supportive | Hands together |
| `ENTERTAINMENT` | Funny | Laughing face |

## Prerequisites

- Publora API key
- LinkedIn account connected via Publora dashboard
- The `postedId` (LinkedIn URN) of the post

## Add a Reaction

**Endpoint:** `POST https://api.publora.com/api/v1/linkedin-reactions`

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN |
| `reactionType` | string | Yes | One of: `LIKE`, `PRAISE`, `EMPATHY`, `INTEREST`, `APPRECIATION`, `ENTERTAINMENT` |
| `platformId` | string | Yes | Your LinkedIn platform ID |

### JavaScript Example - Add "Insightful" Reaction

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reactions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:ugcPost:7429953213384187904',
    reactionType: 'INTEREST',  // Insightful (lightbulb)
    platformId: 'linkedin-ABC123'
  })
});

const data = await response.json();
console.log(data);
// {
//   success: true,
//   reaction: {
//     id: "urn:li:reaction:(urn:li:person:xxx,urn:li:ugcPost:xxx)",
//     reactionType: "INTEREST"
//   },
//   urnTranslated: { from: "urn:li:activity:xxx", to: "urn:li:ugcPost:xxx" }  // Only present if URN was translated
// }
```

> **Note:** The `urnTranslated` field is only present when the provided `postedId` was automatically translated to a different URN format (e.g., from `urn:li:activity:xxx` to `urn:li:ugcPost:xxx`). This helps debug URN format issues.

### Python Example - Add "Like" Reaction

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-reactions',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:ugcPost:7429953213384187904',
        'reactionType': 'LIKE',
        'platformId': 'linkedin-ABC123'
    }
)

print(response.json())
```

### cURL Example - Add "Love" Reaction

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reactions \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:ugcPost:7429953213384187904",
    "reactionType": "EMPATHY",
    "platformId": "linkedin-ABC123"
  }'
```

## Remove a Reaction

**Endpoint:** `DELETE https://api.publora.com/api/v1/linkedin-reactions`

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN |
| `platformId` | string | Yes | Your LinkedIn platform ID |

### JavaScript Example

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reactions', {
  method: 'DELETE',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:ugcPost:7429953213384187904',
    platformId: 'linkedin-ABC123'
  })
});

const data = await response.json();
// { success: true }
```

### cURL Example

```bash
curl -X DELETE https://api.publora.com/api/v1/linkedin-reactions \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:ugcPost:7429953213384187904",
    "platformId": "linkedin-ABC123"
  }'
```

## Common Use Cases

### Auto-engage with industry content

```javascript
// React to multiple posts with "Insightful"
const postIds = [
  'urn:li:ugcPost:123456789',
  'urn:li:ugcPost:987654321',
  'urn:li:ugcPost:456789123'
];

for (const postedId of postIds) {
  await fetch('https://api.publora.com/api/v1/linkedin-reactions', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      postedId,
      reactionType: 'INTEREST',
      platformId: 'linkedin-ABC123'
    })
  });

  // Add delay to avoid rate limits
  await new Promise(r => setTimeout(r, 1000));
}
```

## Important Notes

- **Network visibility:** You can only react to posts visible in your LinkedIn network
- **One reaction per post:** LinkedIn allows only one reaction type per post per user
- **URN format matters:** Use `urn:li:share:xxx` or `urn:li:ugcPost:xxx`, NOT `urn:li:activity:xxx` from LinkedIn URLs
- **Change reaction:** To change your reaction, delete the existing one first, then add the new one

## Getting the Correct URN

LinkedIn URLs use `urn:li:activity:xxx` but the API requires `urn:li:share:xxx` or `urn:li:ugcPost:xxx`.

For posts created via Publora:
```javascript
const response = await fetch('https://api.publora.com/api/v1/get-post/YOUR_POST_GROUP_ID', {
  headers: { 'x-publora-key': 'YOUR_API_KEY' }
});
const postedId = (await response.json()).posts[0].postedId;
```

## Related Guides

- [LinkedIn Comments](linkedin-comments.md) - Comment on posts
- [LinkedIn Mentions](linkedin-mentions.md) - Mention users and companies
- [LinkedIn Platform Guide](../platforms/linkedin.md) - Complete LinkedIn API reference
