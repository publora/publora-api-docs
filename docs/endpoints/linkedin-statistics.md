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

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |
| `queryType` | string | No | Single metric (same options as post statistics) |
| `queryTypes` | string[] or `"ALL"` | No | Multiple metrics at once |
| `aggregation` | string | No | `"TOTAL"` (default) or `"DAILY"` |
| `dateRange` | object | No | `{ start: { year, month, day }, end: { year, month, day } }` |

```json
{
  "platformId": "linkedin-Tz9W5i6ZYG",
  "queryTypes": "ALL",
  "aggregation": "TOTAL"
}
```

### Response (Total Aggregation)

```json
{
  "success": true,
  "metrics": {
    "IMPRESSION": 45230,
    "MEMBERS_REACHED": 22150,
    "RESHARE": 89,
    "REACTION": 1234,
    "COMMENT": 256
  },
  "aggregation": "TOTAL",
  "cached": false
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
    platformId: 'linkedin-Tz9W5i6ZYG',
    queryTypes: 'ALL',
    aggregation: 'TOTAL'
  })
});
const data = await response.json();
console.log(`Total impressions: ${data.metrics.IMPRESSION}`);
console.log(`Total engagement: ${data.metrics.REACTION + data.metrics.COMMENT}`);
```

### Python

```python
response = requests.post(
    'https://api.publora.com/api/v1/linkedin-account-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-Tz9W5i6ZYG',
        'queryTypes': 'ALL',
        'aggregation': 'TOTAL'
    }
)
stats = response.json()
print(f"Total impressions: {stats['metrics']['IMPRESSION']}")
print(f"Total engagement: {stats['metrics']['REACTION'] + stats['metrics']['COMMENT']}")
```

### Node.js (axios)

```javascript
const axios = require('axios');

const client = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
});

// Get post statistics
async function getPostStats(postedId, platformId) {
  const { data } = await client.post('/linkedin-post-statistics', {
    postedId,
    platformId,
    queryTypes: 'ALL'
  });
  return data.metrics;
}

// Get account statistics
async function getAccountStats(platformId) {
  const { data } = await client.post('/linkedin-account-statistics', {
    platformId,
    queryTypes: 'ALL',
    aggregation: 'TOTAL'
  });
  return data.metrics;
}

// Usage
const postMetrics = await getPostStats('urn:li:share:123', 'linkedin-ABC');
const accountMetrics = await getAccountStats('linkedin-ABC');
console.log(`Post impressions: ${postMetrics.IMPRESSION}`);
console.log(`Account total impressions: ${accountMetrics.IMPRESSION}`);
```

### With Error Handling

```javascript
async function getLinkedInStats(platformId, postedId = null) {
  try {
    const endpoint = postedId
      ? '/linkedin-post-statistics'
      : '/linkedin-account-statistics';

    const payload = {
      platformId,
      queryTypes: 'ALL',
      ...(postedId && { postedId }),
      ...(!postedId && { aggregation: 'TOTAL' })
    };

    const response = await fetch(`https://api.publora.com/api/v1${endpoint}`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': process.env.PUBLORA_API_KEY
      },
      body: JSON.stringify(payload)
    });

    const data = await response.json();

    if (!response.ok) {
      switch (response.status) {
        case 400:
          throw new Error(`Invalid request: ${data.error}`);
        case 404:
          throw new Error('LinkedIn account not found. Check your platformId.');
        default:
          throw new Error(data.error || `HTTP ${response.status}`);
      }
    }

    return {
      metrics: data.metrics,
      cached: data.cached,
      engagementRate: calculateEngagementRate(data.metrics)
    };
  } catch (error) {
    console.error('Failed to fetch LinkedIn stats:', error.message);
    throw error;
  }
}

function calculateEngagementRate(metrics) {
  const engagement = metrics.REACTION + metrics.COMMENT + metrics.RESHARE;
  const reach = metrics.MEMBERS_REACHED || metrics.IMPRESSION;
  return reach > 0 ? ((engagement / reach) * 100).toFixed(2) + '%' : '0%';
}
```

```python
import os
import requests

def get_linkedin_stats(platform_id, posted_id=None):
    """Get LinkedIn statistics with error handling."""
    endpoint = 'linkedin-post-statistics' if posted_id else 'linkedin-account-statistics'
    url = f'https://api.publora.com/api/v1/{endpoint}'

    payload = {
        'platformId': platform_id,
        'queryTypes': 'ALL'
    }
    if posted_id:
        payload['postedId'] = posted_id
    else:
        payload['aggregation'] = 'TOTAL'

    try:
        response = requests.post(
            url,
            headers={
                'Content-Type': 'application/json',
                'x-publora-key': os.environ['PUBLORA_API_KEY']
            },
            json=payload
        )

        data = response.json()

        if response.status_code == 400:
            raise ValueError(f"Invalid request: {data.get('error')}")
        if response.status_code == 404:
            raise ValueError('LinkedIn account not found. Check your platformId.')

        response.raise_for_status()

        metrics = data['metrics']
        engagement = metrics['REACTION'] + metrics['COMMENT'] + metrics['RESHARE']
        reach = metrics.get('MEMBERS_REACHED', metrics['IMPRESSION'])
        engagement_rate = (engagement / reach * 100) if reach > 0 else 0

        return {
            'metrics': metrics,
            'cached': data['cached'],
            'engagement_rate': f"{engagement_rate:.2f}%"
        }

    except requests.RequestException as e:
        print(f'Failed to fetch LinkedIn stats: {e}')
        raise


# Usage
stats = get_linkedin_stats('linkedin-Tz9W5i6ZYG')
print(f"Total impressions: {stats['metrics']['IMPRESSION']}")
print(f"Engagement rate: {stats['engagement_rate']}")
```

## Errors

### Post Statistics Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId is required"` | Missing postedId |
| 400 | `"platformId is required"` | Missing platformId |
| 400 | `"Invalid queryType"` | queryType not one of the 5 valid types |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |

### Account Statistics Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"platformId is required"` | Missing platformId |
| 400 | `"Invalid queryType"` | queryType not one of the 5 valid types |
| 400 | `"Invalid aggregation"` | aggregation not TOTAL or DAILY |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Internal server error"` | Server error while fetching statistics |

### Reactions Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId is required"` | Missing postedId |
| 400 | `"platformId is required"` | Missing platformId |
| 400 | `"Invalid reactionType"` | reactionType not one of the 6 valid types |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Internal server error"` | Server error while adding/removing reaction |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
