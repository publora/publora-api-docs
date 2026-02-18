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

Results are cached for improved performance. `cached: true` means the data was served from cache.

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
| 400 | `"Invalid period"` | period is not "lifetime" or "daily" |
| 400 | `"dateRange is required for daily period"` | period is "daily" but dateRange is missing |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to fetch followers statistics"` | LinkedIn API error or internal server error |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
