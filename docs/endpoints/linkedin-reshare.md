# LinkedIn Reshare

Reshare (repost) an existing LinkedIn post to your own feed, with optional commentary. Works for both **personal profile** and **company page** connections.

There are two ways to repost on LinkedIn via Publora:

1. **Immediate reshare** — `POST /linkedin-reshare` (this page). The repost is published right away.
2. **Scheduled reshare** — set `platformSettings.linkedin.repostParentUrn` on [create-post](create-post.md) / [update-post](update-post.md). The repost goes through the normal scheduling pipeline (see [Scheduled Reshares](#scheduled-reshares) below).

## Reshare a Post

```
POST https://api.publora.com/api/v1/linkedin-reshare
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123` — the `linkedin-` prefix is accepted and stripped). Determines who authors the reshare — a personal connection reshares as the member, a company-page connection reshares as the organization. |
| `parent` | string | Yes | URN of the post to reshare. Must be a `urn:li:share:<id>` or `urn:li:ugcPost:<id>` — **not** a `urn:li:activity:` URN (see [parent Format](#parent-format)). |
| `commentary` | string | No | Text added above the reshared post. Max **3000** characters. Defaults to empty (a plain reshare with no added text). |
| `visibility` | string | No | `PUBLIC` or `CONNECTIONS` (case-insensitive), default `PUBLIC`. Organization/company-page reshares must use `PUBLIC`; `CONNECTIONS` is rejected. |

> **Note:** The reshare is authored automatically as the correct entity for the connection — `urn:li:person:*` for a personal profile, `urn:li:organization:*` for a company page. You do not pass the author yourself.

### Response (HTTP 201 Created)

```json
{
  "success": true,
  "reshare": {
    "lifecycleState": "PUBLISHED",
    "id": "urn:li:share:7123456789012345678"
  }
}
```

> **Note:** Creating a reshare returns HTTP **201**, not 200. The `reshare.id` is the URN of the newly created reshare post (read from LinkedIn's `x-restli-id` response header); the rest of the `reshare` object is the LinkedIn API response body.

### Examples

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reshare', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-Tz9W5i6ZYG',
    parent: 'urn:li:share:7123456789012345678',
    commentary: 'Great read — sharing with my network!',
    visibility: 'PUBLIC'
  })
});
const data = await response.json();
console.log(data.reshare.id); // urn:li:share:...
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-reshare',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-Tz9W5i6ZYG',
        'parent': 'urn:li:share:7123456789012345678',
        'commentary': 'Great read — sharing with my network!'
    }
)
print(response.json()['reshare']['id'])
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reshare \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platformId": "linkedin-Tz9W5i6ZYG",
    "parent": "urn:li:share:7123456789012345678",
    "commentary": "Great read — sharing with my network!",
    "visibility": "PUBLIC"
  }'
```

#### Node.js (axios)

```javascript
const axios = require('axios');

await axios.post(
  'https://api.publora.com/api/v1/linkedin-reshare',
  {
    platformId: 'linkedin-Tz9W5i6ZYG',
    parent: 'urn:li:ugcPost:7123456789012345678'
  },
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
```

### parent Format

Use `urn:li:share:xxx` or `urn:li:ugcPost:xxx` — **not** `urn:li:activity:xxx`.

The `urn:li:activity:` URN is what appears in LinkedIn post URLs (e.g. `linkedin.com/feed/update/urn:li:activity:123`), but it is rejected by this endpoint with a 400 error.

To get the correct URN:
- For posts created via Publora, use the `postedId` field from the [get-post](get-post.md) response
- The activity ID and share ID are typically the same number — try replacing `urn:li:activity:` with `urn:li:share:` (e.g. `urn:li:activity:7451373349668282369` → `urn:li:share:7451373349668282369`)

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"platformId and parent are required"` | Missing `platformId` or `parent` |
| 400 | `"platformId must be a string"` | `platformId` is not a string |
| 400 | `"platformId cannot be empty"` | `platformId` is blank/whitespace |
| 400 | `"parent must be a string"` | `parent` is not a string |
| 400 | `"parent cannot be empty"` | `parent` is blank/whitespace |
| 400 | `"parent must be a valid LinkedIn post URN (urn:li:share:<id> or urn:li:ugcPost:<id>)"` | `parent` is not a `share`/`ugcPost` URN |
| 400 | `"commentary must be a string"` | `commentary` is not a string |
| 400 | `"commentary cannot exceed 3000 characters"` | `commentary` is longer than 3000 characters |
| 400 | `"visibility must be a string"` | `visibility` is not a string |
| 400 | `"visibility must be one of: PUBLIC, CONNECTIONS"` | `visibility` is not a valid value |
| 400 | `"LinkedIn organization reposts cannot use CONNECTIONS visibility; choose PUBLIC"` | A company-page connection requested member-only visibility |
| 400 | `"Invalid platformId"` | `platformId` format is invalid |
| 400 | `"LinkedIn company connection is missing organizationId. Please reconnect the LinkedIn page."` | Company-page connection has no stored organization id — reconnect the page |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"LINKEDIN_TOKEN_EXPIRED"` | The LinkedIn access token is expired/invalid — reconnect the account |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that `platformId` for this user |
| 500 | `"Failed to create LinkedIn reshare"` | Server error while creating the reshare |

> **Note:** The error response for a failed LinkedIn call includes a `details` **string** with the underlying LinkedIn error message: `{ "error": "Failed to create LinkedIn reshare", "details": "<LinkedIn error message>" }` (unlike the reactions endpoint, there is no separate `status` field in the body).

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403 if the post's author disabled resharing, 429 on rate limits), so you may receive error codes other than those listed above.

## Scheduled Reshares

To schedule a repost for a future time instead of publishing immediately, use the normal scheduling pipeline with LinkedIn platform settings:

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'My commentary on this great post.',
    platforms: ['linkedin-Tz9W5i6ZYG'],
    scheduledTime: '2026-08-01T14:00:00.000Z',
    platformSettings: {
      linkedin: {
        repostParentUrn: 'urn:li:share:7123456789012345678',
        repostVisibility: 'PUBLIC'
      }
    }
  })
});
```

The post `content` becomes the reshare commentary. **Media is not allowed on repost groups.** See [Create Post → LinkedIn Repost Settings](create-post.md#linkedin-repost-settings) for the full field reference and validation rules.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
