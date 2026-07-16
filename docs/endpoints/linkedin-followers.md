# LinkedIn Followers

Retrieve follower statistics for your LinkedIn account, including total follower count and daily growth data.

## Endpoint

```
POST https://api.publora.com/api/v1/linkedin-followers
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |
| `period` | string | No | `"lifetime"` (default) or `"daily"` |
| `dateRange` | object | Conditional | Required when `period="daily"`. Contains `start` and `end` date objects. |

### Date Range Format

> **Note:** The `period` parameter is **case-insensitive** — e.g., `"daily"`, `"Daily"`, and `"DAILY"` are all accepted.

When using `period="daily"`, you must provide a `dateRange` object:

```json
{
  "dateRange": {
    "start": { "year": 2024, "month": 1, "day": 1 },
    "end": { "year": 2024, "month": 1, "day": 31 }
  }
}
```

## Response (Lifetime)

When `period="lifetime"` (or omitted), returns the total follower count:

```json
{
  "success": true,
  "followersCount": 1234,
  "cached": false
}
```

## Response (Daily)

When `period="daily"`, returns daily follower growth data:

```json
{
  "success": true,
  "data": [
    { "date": "2024-01-15", "count": 10 },
    { "date": "2024-01-16", "count": 15 },
    { "date": "2024-01-17", "count": 8 }
  ],
  "totalGrowth": 33,
  "cached": false
}
```

Results are cached for 2 hours. `cached: true` means the data was served from cache.

## Examples

### Get total follower count

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-followers', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-Tz9W5i6ZYG'
  })
});
const data = await response.json();
console.log(`Total followers: ${data.followersCount}`);
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-followers',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-Tz9W5i6ZYG'
    }
)
data = response.json()
print(f"Total followers: {data['followersCount']}")
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-followers \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platformId": "linkedin-Tz9W5i6ZYG"
  }'
```

### Get daily follower growth

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-followers', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-Tz9W5i6ZYG',
    period: 'daily',
    dateRange: {
      start: { year: 2024, month: 1, day: 1 },
      end: { year: 2024, month: 1, day: 31 }
    }
  })
});
const data = await response.json();
console.log(`Total growth: ${data.totalGrowth} new followers`);
data.data.forEach(day => {
  console.log(`${day.date}: +${day.count} followers`);
});
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-followers',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-Tz9W5i6ZYG',
        'period': 'daily',
        'dateRange': {
            'start': {'year': 2024, 'month': 1, 'day': 1},
            'end': {'year': 2024, 'month': 1, 'day': 31}
        }
    }
)
data = response.json()
print(f"Total growth: {data['totalGrowth']} new followers")
for day in data['data']:
    print(f"{day['date']}: +{day['count']} followers")
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-followers \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platformId": "linkedin-Tz9W5i6ZYG",
    "period": "daily",
    "dateRange": {
      "start": { "year": 2024, "month": 1, "day": 1 },
      "end": { "year": 2024, "month": 1, "day": 31 }
    }
  }'
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"platformId is required"` | Missing platformId in request body |
| 400 | `"platformId must be a string"` | platformId is not a string type |
| 400 | `"Invalid platformId"` | platformId is a string but has an invalid format (empty or unrecognized) |
| 400 | `"period must be a string"` | period parameter is not a string type |
| 400 | `"Invalid period. Must be one of: lifetime, daily"` | period is not "lifetime" or "daily" |
| 400 | `"dateRange is required for daily period"` | period is "daily" but dateRange is missing |
| 400 | `"dateRange must be an object"` | dateRange is not a plain object (e.g., it is null, an array, or a non-object type) |
| 400 | `"dateRange must have both 'start' and 'end' objects"` | dateRange provided but missing start or end |
| 400 | `"dateRange.start and dateRange.end must have: year, month, day"` | dateRange start/end missing required fields |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to fetch LinkedIn followers statistics"` | LinkedIn API error or internal server error |

> **Note:** The "Invalid platformId" error only triggers for empty or unrecognized string values. Non-string values are caught earlier by the `"platformId must be a string"` check. Any non-empty string passes validation here. By contrast, `create-post` parses a compound key at the first `-`: its prefix must be lowercase letters, and its non-empty ID rejects whitespace, `/`, `?`, and `#` (colons are allowed). This applies to followers, reactions, comments, and profile summary endpoints — but **not** statistics endpoints, which use inline extraction and never return this error.

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403, 429), so you may receive error codes other than those listed above.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
