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

> **Note:** The `queryTypes` (plural) and `aggregation` parameters are **case-insensitive** — e.g., `"impression"`, `"Impression"`, and `"IMPRESSION"` are all accepted. However, `queryType` (single metric) **must be UPPERCASE** — only `"IMPRESSION"` is valid, not `"impression"` or `"Impression"`.

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
  "cachedMetrics": ["IMPRESSION", "REACTION"],
  "cached": true
}
```

The `cachedMetrics` field lists which individual metrics were served from cache. It is only present when at least one metric was served from cache (i.e., the array is non-empty). Results are cached for 2 hours. `cached: true` means at least one metric was served from cache.

### Partial Failure (Multiple Metrics)

When requesting multiple metrics, some may succeed while others fail. The response includes an `errors` array:

```json
{
  "success": false,
  "metrics": {
    "IMPRESSION": 15420,
    "MEMBERS_REACHED": null,
    "REACTION": 456,
    "COMMENT": 89,
    "RESHARE": null
  },
  "errors": [
    { "metric": "MEMBERS_REACHED", "error": "Failed to fetch metric from LinkedIn" },
    { "metric": "RESHARE", "error": "Failed to fetch metric from LinkedIn" }
  ],
  "cached": false
}
```

Failed metrics are included in the `metrics` object with `null` values (they are not omitted). The error message is always `"Failed to fetch metric from LinkedIn"` regardless of the underlying cause. Check the `errors` array to identify which metrics failed.

> **Note:** A metric value may be `null` in the `metrics` object without a corresponding entry in the `errors` array. This occurs when LinkedIn returns a `-1` count (interpreted as unavailable data), as opposed to a fetch failure which produces both a `null` metric and an `errors` entry.

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

> **Note:** The `queryTypes` (plural) and `aggregation` parameters are **case-insensitive** — e.g., `"total"`, `"Total"`, and `"TOTAL"` are all accepted. However, `queryType` (single metric) **must be UPPERCASE** — only `"IMPRESSION"` is valid, not `"impression"` or `"Impression"`.

```json
{
  "platformId": "linkedin-Tz9W5i6ZYG",
  "queryTypes": "ALL",
  "aggregation": "TOTAL"
}
```

### Response (Single Metric)

When using `queryType` (single metric), the account statistics response includes the raw data and aggregation type. The `data` field type depends on the aggregation:

- **TOTAL aggregation:** `data` is a **number** (the aggregated total), or `null` when LinkedIn returns `-1` (indicating data is unavailable for that metric).
- **DAILY aggregation:** `data` is an **array of objects**, each with `count` and `dateRange` fields.

#### TOTAL aggregation example

```json
{
  "success": true,
  "data": 15420,
  "aggregation": "TOTAL",
  "cached": false
}
```

#### DAILY aggregation example

```json
{
  "success": true,
  "data": [
    { "count": 1200, "dateRange": { "start": { "year": 2024, "month": 1, "day": 15 }, "end": { "year": 2024, "month": 1, "day": 16 } } },
    { "count": 980, "dateRange": { "start": { "year": 2024, "month": 1, "day": 16 }, "end": { "year": 2024, "month": 1, "day": 17 } } }
  ],
  "aggregation": "DAILY",
  "cached": false
}
```

### Response (Multiple Metrics — Total Aggregation)

When using `queryTypes` (multiple metrics), the response uses the same format as post statistics but also includes the `aggregation` field:

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
  "cachedMetrics": ["IMPRESSION"],
  "cached": true
}
```

### Response (Daily Aggregation)

When `aggregation` is `"DAILY"`, each metric value is an array of daily data points instead of a single number:

```json
{
  "success": true,
  "metrics": {
    "IMPRESSION": [
      { "count": 1200, "dateRange": { "start": { "year": 2024, "month": 1, "day": 15 }, "end": { "year": 2024, "month": 1, "day": 16 } } },
      { "count": 980, "dateRange": { "start": { "year": 2024, "month": 1, "day": 16 }, "end": { "year": 2024, "month": 1, "day": 17 } } }
    ]
  },
  "aggregation": "DAILY",
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
| 400 | `"postedId and platformId are required"` | Missing postedId or platformId |
| 400 | `"postedId and platformId must be strings"` | postedId or platformId is not a string type |
| 400 | `"queryType or queryTypes is required"` | Neither queryType nor queryTypes provided |
| 400 | `"Invalid queryType. Must be one of: IMPRESSION, MEMBERS_REACHED, RESHARE, REACTION, COMMENT"` | queryType not one of the 5 valid types |
| 400 | `"Invalid queryTypes: ${invalidTypes}. Must be one of: IMPRESSION, MEMBERS_REACHED, RESHARE, REACTION, COMMENT, or \"ALL\""` | queryTypes contains invalid values |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to fetch metric from LinkedIn"` | Server error while fetching a single metric (queryType) — LinkedIn returned null data |
| 500 | `"Failed to fetch LinkedIn post statistics"` | Catch-all server error from outer try/catch |

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403, 429), so you may receive error codes other than those listed above.

### Account Statistics Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"platformId is required"` | Missing platformId |
| 400 | `"platformId must be a string"` | platformId is not a string type |
| 400 | `"queryType or queryTypes is required"` | Neither queryType nor queryTypes provided |
| 400 | `"Invalid queryType. Must be one of: IMPRESSION, MEMBERS_REACHED, RESHARE, REACTION, COMMENT"` | queryType not one of the 5 valid types |
| 400 | `"Invalid queryTypes: ${invalidTypes}. Must be one of: IMPRESSION, MEMBERS_REACHED, RESHARE, REACTION, COMMENT, or \"ALL\""` | queryTypes contains invalid values |
| 400 | `"aggregation must be a string"` | aggregation parameter was provided but is not a string type |
| 400 | `"Invalid aggregation. Must be one of: DAILY, TOTAL"` | aggregation not TOTAL or DAILY |
| 400 | `"dateRange must be an object"` | dateRange parameter was provided but is not an object type |
| 400 | `"dateRange must have both 'start' and 'end' objects"` | dateRange provided but missing start or end |
| 400 | `"dateRange.start and dateRange.end must have: year, month, day"` | dateRange start/end missing required fields |
| 400 | `"MEMBERS_REACHED with DAILY aggregation is not supported by LinkedIn API"` | Unsupported metric/aggregation combination |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to fetch metric from LinkedIn"` | Single-metric request (queryType) returned null data from LinkedIn |
| 500 | `"Failed to fetch LinkedIn account statistics"` | Catch-all server error from outer try/catch |

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403, 429), so you may receive error codes other than those listed above.

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
