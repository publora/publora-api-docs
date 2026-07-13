# LinkedIn Reactions

Add or remove reactions on LinkedIn posts programmatically.

## Add Reaction

```
POST https://api.publora.com/api/v1/linkedin-reactions
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn URN (e.g., `urn:li:share:123`, `urn:li:ugcPost:456`, `urn:li:activity:789`) |
| `reactionType` | string | Yes | One of: `LIKE`, `PRAISE`, `EMPATHY`, `INTEREST`, `APPRECIATION`, `ENTERTAINMENT` |
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |

### Reaction Types

| Type | Emoji | Description |
|------|-------|-------------|
| `LIKE` | 👍 | Thumbs up |
| `PRAISE` | 🔥 | Celebrate / Fire |
| `EMPATHY` | ❤️ | Love / Heart |
| `INTEREST` | 💡 | Insightful / Light bulb |
| `APPRECIATION` | 👏 | Support / Clapping hands |
| `ENTERTAINMENT` | 😄 | Funny / Laughing |

> **Note:** The `reactionType` parameter is **case-insensitive** — e.g., `"like"`, `"Like"`, and `"LIKE"` are all accepted.

### Response (HTTP 201 Created)

```json
{
  "success": true,
  "reaction": { ... },
  "urnTranslated": {
    "from": "urn:li:activity:7123456789012345678",
    "to": "urn:li:ugcPost:7123456789012345678"
  }
}
```

> **Note:** Creating a reaction returns HTTP **201**, not 200. The `urnTranslated` field is an object `{ from, to }` showing the original and canonical URN when translation occurred (e.g., `urn:li:activity:*` translated to `urn:li:ugcPost:*`). It is absent/undefined when no translation was needed.

### Examples

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reactions', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    reactionType: 'LIKE',
    platformId: 'linkedin-Tz9W5i6ZYG'
  })
});
const data = await response.json();
console.log(data.success); // true
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-reactions',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7123456789012345678',
        'reactionType': 'PRAISE',
        'platformId': 'linkedin-Tz9W5i6ZYG'
    }
)
print(response.json()['success'])
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-reactions \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7123456789012345678",
    "reactionType": "LIKE",
    "platformId": "linkedin-Tz9W5i6ZYG"
  }'
```

#### Node.js (axios)

```javascript
const axios = require('axios');

await axios.post(
  'https://api.publora.com/api/v1/linkedin-reactions',
  {
    postedId: 'urn:li:share:7123456789012345678',
    reactionType: 'EMPATHY',
    platformId: 'linkedin-Tz9W5i6ZYG'
  },
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
```

---

## Remove Reaction

```
DELETE https://api.publora.com/api/v1/linkedin-reactions
```

### Request Body or Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn URN |
| `platformId` | string | Yes | LinkedIn connection ID |

> **Note:** Unlike creating a reaction, deleting does not require `reactionType`.

### Response

```json
{
  "success": true,
  "reaction": null,
  "urnTranslated": {
    "from": "urn:li:activity:7123456789012345678",
    "to": "urn:li:ugcPost:7123456789012345678"
  }
}
```

The `reaction` field contains the LinkedIn API response data, or `null`. The `urnTranslated` field follows the same format as the create response (object or absent).

### Examples

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-reactions', {
  method: 'DELETE',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    platformId: 'linkedin-Tz9W5i6ZYG'
  })
});
```

#### Python (requests)

```python
response = requests.delete(
    'https://api.publora.com/api/v1/linkedin-reactions',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7123456789012345678',
        'platformId': 'linkedin-Tz9W5i6ZYG'
    }
)
```

#### cURL

```bash
curl -X DELETE "https://api.publora.com/api/v1/linkedin-reactions?postedId=urn:li:share:7123456789012345678&platformId=linkedin-Tz9W5i6ZYG" \
  -H "x-publora-key: YOUR_API_KEY"
```

## Errors

### Create Reaction Errors (POST)

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId, reactionType, and platformId are required"` | Missing one or more required fields |
| 400 | `"platformId must be a string"` | platformId is not a string type |
| 400 | `"postedId must be a valid LinkedIn URN"` | Not a valid LinkedIn URN format |
| 400 | `"reactionType must be one of: LIKE, PRAISE, EMPATHY, INTEREST, APPRECIATION, ENTERTAINMENT"` | Not one of the 6 valid types |
| 400 | `"Invalid platformId"` | platformId format is invalid |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 409 | `"REACTION_ALREADY_EXISTS"` | A reaction by this account already exists on the post (any type). LinkedIn keys reactions by `(actor, entity)`, so delete the existing reaction before applying a different one. |
| 500 | `"Failed to create LinkedIn reaction"` | Server error while adding reaction |

> **Note:** The 500 error response includes `details` and `status` fields: `{ "error": "Failed to create LinkedIn reaction", "details": { ... }, "status": <HTTP status> }`. The `details` field contains the full LinkedIn API error response object, whose structure depends on what LinkedIn returns (e.g., it may include `message`, `status`, `serviceErrorCode`, etc.). The `status` field reflects the HTTP status code returned by the LinkedIn API.

> **Note:** POSTing a reaction that already exists returns **409** with body `{ "error": "REACTION_ALREADY_EXISTS", "message": "A reaction already exists on this post. Delete it before applying '<TYPE>'.", "requestedReactionType": "<TYPE>" }`. Publora cannot read your existing reaction type back (LinkedIn scope limitation), so the conflict is surfaced rather than silently overwritten. To change reaction type, [remove the reaction](#remove-reaction) first, then POST the new one.

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403, 429), so you may receive error codes other than those listed above.

### Delete Reaction Errors (DELETE)

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId and platformId are required"` | Missing postedId or platformId |
| 400 | `"platformId must be a string"` | platformId is not a string type |
| 400 | `"postedId must be a valid LinkedIn URN"` | Not a valid LinkedIn URN format |
| 400 | `"Invalid platformId"` | platformId format is invalid |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to delete LinkedIn reaction"` | Server error while removing reaction |

> **Note:** The 500 error response includes `details` and `status` fields: `{ "error": "Failed to delete LinkedIn reaction", "details": { ... }, "status": <HTTP status> }`. The `details` field contains the full LinkedIn API error response object, whose structure depends on what LinkedIn returns (e.g., it may include `message`, `status`, `serviceErrorCode`, etc.). The `status` field reflects the HTTP status code returned by the LinkedIn API.

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403, 429), so you may receive error codes other than those listed above.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
