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

### Response

```json
{
  "success": true,
  "reaction": { ... }
}
```

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

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId is required"` | Missing postedId |
| 400 | `"Invalid postedId"` | Not a valid LinkedIn URN format |
| 400 | `"Invalid reactionType"` | Not one of the 6 valid types |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
