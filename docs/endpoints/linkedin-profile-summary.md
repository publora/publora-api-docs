# LinkedIn Profile Summary

Retrieve a summary of your LinkedIn profile analytics including follower counts and aggregated post engagement metrics.

## Endpoint

```
POST https://api.publora.com/api/v1/linkedin-profile-summary
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-{id}`) |
| `dateRange` | object | No | Date range filter for metrics |
| `dateRange.start` | object | No | Start date: `{ year, month, day }` |
| `dateRange.end` | object | No | End date: `{ year, month, day }` |

## Response

### Success Response

```json
{
  "success": true,
  "profile": {
    "followers": {
      "total": 5000,
      "periodGrowth": 150
    },
    "posts": {
      "totalImpressions": 25000,
      "totalReactions": 500,
      "totalComments": 75,
      "totalReshares": 30
    }
  }
}
```

### Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `profile.followers.total` | integer | Total follower count |
| `profile.followers.periodGrowth` | integer | Follower growth during date range (only present when `dateRange` is provided) |
| `profile.posts.totalImpressions` | integer | Total post impressions |
| `profile.posts.totalReactions` | integer | Total reactions across all posts |
| `profile.posts.totalComments` | integer | Total comments across all posts |
| `profile.posts.totalReshares` | integer | Total reshares across all posts |

### Partial Failure Response

When some data was fetched successfully but other requests failed:

```json
{
  "success": false,
  "partialData": true,
  "profile": {
    "followers": {
      "total": 5000
    },
    "posts": {
      "totalImpressions": 25000,
      "totalReactions": 500,
      "totalComments": 75,
      "totalReshares": 30
    }
  },
  "errors": [
    { "metric": "followers", "error": "Failed to fetch followers" }
  ]
}
```

The `partialData: true` flag indicates some metrics are available despite the failure. Check the `errors` array for details on what failed. Each error object contains:
- `metric`: The metric that failed to fetch
- `error`: The error message describing what went wrong

## Examples

### Get full profile summary

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-profile-summary', {
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
console.log(`Followers: ${data.profile.followers.total}`);
console.log(`Total Impressions: ${data.profile.posts.totalImpressions}`);
console.log(`Total Engagement: ${data.profile.posts.totalReactions + data.profile.posts.totalComments}`);
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-profile-summary',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-Tz9W5i6ZYG'
    }
)
data = response.json()
profile = data['profile']
print(f"Followers: {profile['followers']['total']}")
print(f"Total Impressions: {profile['posts']['totalImpressions']}")
print(f"Total Reactions: {profile['posts']['totalReactions']}")
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-profile-summary \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platformId": "linkedin-Tz9W5i6ZYG"
  }'
```

### Get profile summary with date range

Track follower growth and metrics for a specific time period:

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-profile-summary', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-Tz9W5i6ZYG',
    dateRange: {
      start: { year: 2026, month: 1, day: 1 },
      end: { year: 2026, month: 1, day: 31 }
    }
  })
});
const data = await response.json();
console.log(`Follower growth in January: ${data.profile.followers.periodGrowth}`);
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-profile-summary',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-Tz9W5i6ZYG',
        'dateRange': {
            'start': {'year': 2026, 'month': 1, 'day': 1},
            'end': {'year': 2026, 'month': 1, 'day': 31}
        }
    }
)
data = response.json()
print(f"Follower growth: {data['profile']['followers']['periodGrowth']}")
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-profile-summary \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platformId": "linkedin-Tz9W5i6ZYG",
    "dateRange": {
      "start": { "year": 2026, "month": 1, "day": 1 },
      "end": { "year": 2026, "month": 1, "day": 31 }
    }
  }'
```

### Handle partial failures

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-profile-summary', {
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

if (data.partialData) {
  console.warn('Some data could not be fetched:', data.errors);
  // Still use available data
  if (data.profile.posts) {
    console.log(`Impressions: ${data.profile.posts.totalImpressions}`);
  }
} else if (data.success) {
  console.log(`Full profile data received`);
}
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"platformId is required"` | Missing platformId in request body |
| 400 | `"Invalid dateRange format"` | dateRange object has invalid structure |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Unexpected error"` | Server error |
| 502 | `"All requests failed"` | Unable to fetch any data from LinkedIn |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
