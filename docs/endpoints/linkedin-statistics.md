# LinkedIn Post Statistics

Retrieve analytics for your LinkedIn posts: impressions, reach, reactions, comments, and reshares.

## Endpoint

```
POST https://api.publora.com/api/v1/linkedin-post-statistics
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

## Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:123456789`) or share ID |
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |
| `queryType` | string | No | Single metric: `IMPRESSION`, `MEMBERS_REACHED`, `RESHARE`, `REACTION`, `COMMENT` |
| `queryTypes` | string[] or `"ALL"` | No | Multiple metrics at once. Use `"ALL"` for all 5 metrics. |

Use either `queryType` (single) or `queryTypes` (multiple), not both.

## Available Metrics

| Metric | Description |
|--------|-------------|
| `IMPRESSION` | Total number of times the post was displayed |
| `MEMBERS_REACHED` | Unique LinkedIn members who saw the post |
| `RESHARE` | Number of times the post was reposted |
| `REACTION` | Total reactions (LIKE, PRAISE, EMPATHY, INTEREST, APPRECIATION, ENTERTAINMENT) |
| `COMMENT` | Total number of comments |

## Response (Single Metric)

```json
{
  "success": true,
  "count": 1542,
  "cached": false
}
```

## Response (Multiple Metrics)

```json
{
  "success": true,
  "metrics": {
    "IMPRESSION": 15420,
    "MEMBERS_REACHED": 8234,
    "RESHARE": 23,
    "REACTION": 456,
    "COMMENT": 89
  },
  "cached": false
}
```

Results are cached for 30 minutes. `cached: true` means the data was served from cache.

## Examples

### Get all metrics for a post

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    platformId: 'linkedin-Tz9W5i6ZYG',
    queryTypes: 'ALL'
  })
});
const data = await response.json();
console.log(`Impressions: ${data.metrics.IMPRESSION}`);
console.log(`Reactions: ${data.metrics.REACTION}`);
console.log(`Comments: ${data.metrics.COMMENT}`);
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-post-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7123456789012345678',
        'platformId': 'linkedin-Tz9W5i6ZYG',
        'queryTypes': 'ALL'
    }
)
metrics = response.json()['metrics']
print(f"Impressions: {metrics['IMPRESSION']}")
print(f"Reach: {metrics['MEMBERS_REACHED']}")
print(f"Engagement: {metrics['REACTION'] + metrics['COMMENT'] + metrics['RESHARE']}")
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-post-statistics \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7123456789012345678",
    "platformId": "linkedin-Tz9W5i6ZYG",
    "queryTypes": "ALL"
  }'
```

### Get a single metric

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    platformId: 'linkedin-Tz9W5i6ZYG',
    queryType: 'IMPRESSION'
  })
});
const data = await response.json();
console.log(`Impressions: ${data.count}`); // 1542
```

### Get specific metrics

```python
response = requests.post(
    'https://api.publora.com/api/v1/linkedin-post-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7123456789012345678',
        'platformId': 'linkedin-Tz9W5i6ZYG',
        'queryTypes': ['IMPRESSION', 'REACTION', 'COMMENT']
    }
)
```

## LinkedIn Account Statistics

Get aggregated metrics across all published LinkedIn posts for an account.

### Endpoint

```
POST https://api.publora.com/api/v1/linkedin-account-statistics
```

### Request Body

```json
{
  "platformId": "linkedin-Tz9W5i6ZYG"
}
```

### Response

```json
{
  "success": true,
  "aggregatedMetrics": {
    "totalImpressions": 45230,
    "totalMembersReached": 22150,
    "totalReshares": 89,
    "totalReactions": 1234,
    "totalComments": 256
  },
  "posts": [
    {
      "postedId": "urn:li:share:7123456789",
      "metrics": {
        "IMPRESSION": 15420,
        "MEMBERS_REACHED": 8234,
        "RESHARE": 23,
        "REACTION": 456,
        "COMMENT": 89
      }
    }
  ]
}
```

### JavaScript

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-account-statistics', {
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
console.log(`Total impressions: ${data.aggregatedMetrics.totalImpressions}`);
console.log(`Total engagement: ${data.aggregatedMetrics.totalReactions + data.aggregatedMetrics.totalComments}`);
```

### Python

```python
response = requests.post(
    'https://api.publora.com/api/v1/linkedin-account-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={'platformId': 'linkedin-Tz9W5i6ZYG'}
)
stats = response.json()
print(f"Total impressions: {stats['aggregatedMetrics']['totalImpressions']}")
print(f"Posts analyzed: {len(stats['posts'])}")
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId is required"` | Missing postedId |
| 400 | `"platformId is required"` | Missing platformId |
| 400 | `"Invalid queryType"` | queryType not one of the 5 valid types |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
