# LinkedIn Analytics Guide

Track the performance of your LinkedIn posts with Publora's analytics endpoints.

## Overview

Publora provides analytics for LinkedIn posts, including:
- **Post-level metrics:** impressions, reach, reactions, comments, reshares
- **Account-level metrics:** aggregated impressions, reactions, comments, reshares, and reach

> **Note:** Analytics are currently available for LinkedIn only. Other platforms are planned for future releases.

## Available Metrics

| Metric | Description |
|--------|-------------|
| `IMPRESSION` | Total number of times the post was displayed |
| `MEMBERS_REACHED` | Unique LinkedIn members who saw the post |
| `RESHARE` | Number of times the post was reposted |
| `REACTION` | Total reactions (LIKE, PRAISE, EMPATHY, INTEREST, APPRECIATION, ENTERTAINMENT) |
| `COMMENT` | Total number of comments |

## Get Post Statistics

### Endpoint

```
POST /api/v1/linkedin-post-statistics
```

### Request

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (e.g., `urn:li:share:7123456789012345678`) |
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |
| `queryType` | string | No | Single metric: `IMPRESSION`, `MEMBERS_REACHED`, `RESHARE`, `REACTION`, `COMMENT` |
| `queryTypes` | string[] or `"ALL"` | No | Multiple metrics at once. Use `"ALL"` for all 5 metrics. |

Use either `queryType` (single) or `queryTypes` (multiple), not both.

### Get All Metrics

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    platformId: 'linkedin-ABC123DEF',
    queryTypes: 'ALL'
  })
});

const data = await response.json();
console.log('Impressions:', data.metrics.IMPRESSION);
console.log('Reach:', data.metrics.MEMBERS_REACHED);
console.log('Engagement:', data.metrics.REACTION + data.metrics.COMMENT + data.metrics.RESHARE);
```

**Python (requests)**

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
        'platformId': 'linkedin-ABC123DEF',
        'queryTypes': 'ALL'
    }
)

data = response.json()
metrics = data['metrics']
print(f"Impressions: {metrics['IMPRESSION']}")
print(f"Reach: {metrics['MEMBERS_REACHED']}")
print(f"Engagement: {metrics['REACTION'] + metrics['COMMENT'] + metrics['RESHARE']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-post-statistics \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "postedId": "urn:li:share:7123456789012345678",
    "platformId": "linkedin-ABC123DEF",
    "queryTypes": "ALL"
  }'
```

### Response (All Metrics)

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

### Get a Single Metric

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    platformId: 'linkedin-ABC123DEF',
    queryType: 'IMPRESSION'
  })
});

const data = await response.json();
console.log(`Impressions: ${data.count}`); // Single metric returns { success, count, cached }
```

### Response (Single Metric)

```json
{
  "success": true,
  "count": 15420,
  "cached": false
}
```

## Get Account Statistics

### Endpoint

```
POST /api/v1/linkedin-account-statistics
```

### Request

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |
| `queryType` | string | No | Single metric (same options as post statistics) |
| `queryTypes` | string[] or `"ALL"` | No | Multiple metrics at once |
| `aggregation` | string | No | `"TOTAL"` (default) or `"DAILY"` |
| `dateRange` | object | No | `{ start: { year, month, day }, end: { year, month, day } }` |

> **Note:** `MEMBERS_REACHED` with `DAILY` aggregation is not supported by the LinkedIn API.

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-account-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-ABC123DEF',
    queryTypes: 'ALL',
    aggregation: 'TOTAL'
  })
});

const data = await response.json();
console.log('Total impressions:', data.metrics.IMPRESSION);
console.log('Total engagement:', data.metrics.REACTION + data.metrics.COMMENT);
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-account-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'platformId': 'linkedin-ABC123DEF',
        'queryTypes': 'ALL',
        'aggregation': 'TOTAL'
    }
)

data = response.json()
print(f"Total impressions: {data['metrics']['IMPRESSION']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-account-statistics \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformId": "linkedin-ABC123DEF",
    "queryTypes": "ALL",
    "aggregation": "TOTAL"
  }'
```

### Response (Total Aggregation, Multiple Metrics)

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

### Daily Aggregation with Date Range

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-account-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformId: 'linkedin-ABC123DEF',
    queryType: 'IMPRESSION',
    aggregation: 'DAILY',
    dateRange: {
      start: { year: 2026, month: 1, day: 1 },
      end: { year: 2026, month: 1, day: 31 }
    }
  })
});

const data = await response.json();
// Returns array of daily values
for (const entry of data.data) {
  console.log(`${JSON.stringify(entry.dateRange)}: ${entry.count} impressions`);
}
```

### Response (Daily Aggregation, Single Metric)

```json
{
  "success": true,
  "data": [
    {
      "count": 150,
      "dateRange": { "start": { "year": 2026, "month": 1, "day": 1 }, "end": { "year": 2026, "month": 1, "day": 1 } }
    },
    {
      "count": 220,
      "dateRange": { "start": { "year": 2026, "month": 1, "day": 2 }, "end": { "year": 2026, "month": 1, "day": 2 } }
    }
  ],
  "aggregation": "DAILY",
  "cached": false
}
```

## Finding the Post URN

When you create a post via Publora, the response includes the platform-specific post ID. For LinkedIn, this is the URN.

### From Post Creation Response

```javascript
const createResponse = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'My LinkedIn post',
    platforms: ['linkedin-ABC123DEF']
  })
});

const result = await createResponse.json();
const postGroupId = result.postGroupId;

// Wait for post to be published, then get details
const postDetails = await fetch(`https://api.publora.com/api/v1/get-post/${postGroupId}`, {
  headers: { 'x-publora-key': 'YOUR_API_KEY' }
}).then(r => r.json());

// Find the LinkedIn postedId
const linkedinPost = postDetails.posts.find(p => p.platform === 'linkedin');
const postedId = linkedinPost.postedId; // e.g., "urn:li:share:7123456789012345678"
```

## Analytics Dashboard Example

Build a simple analytics tracker for your LinkedIn posts.

### JavaScript Implementation

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

async function getPostAnalytics(platformId, postedIds) {
  const results = [];

  for (const postedId of postedIds) {
    const response = await fetch(`${BASE_URL}/linkedin-post-statistics`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': PUBLORA_API_KEY
      },
      body: JSON.stringify({ platformId, postedId, queryTypes: 'ALL' })
    });

    const data = await response.json();
    const metrics = data.metrics;
    results.push({
      postedId,
      ...metrics,
      totalEngagement: metrics.REACTION + metrics.COMMENT + metrics.RESHARE
    });
  }

  return results;
}

async function generateReport(platformId, postedIds) {
  console.log('=== LinkedIn Analytics Report ===\n');

  const postStats = await getPostAnalytics(platformId, postedIds);

  const totals = postStats.reduce((acc, post) => ({
    IMPRESSION: acc.IMPRESSION + post.IMPRESSION,
    MEMBERS_REACHED: acc.MEMBERS_REACHED + post.MEMBERS_REACHED,
    REACTION: acc.REACTION + post.REACTION,
    COMMENT: acc.COMMENT + post.COMMENT,
    RESHARE: acc.RESHARE + post.RESHARE
  }), { IMPRESSION: 0, MEMBERS_REACHED: 0, REACTION: 0, COMMENT: 0, RESHARE: 0 });

  console.log('Post Performance:');
  console.log('-'.repeat(60));

  postStats.forEach((post, i) => {
    console.log(`\nPost ${i + 1}: ${post.postedId}`);
    console.log(`  Impressions: ${post.IMPRESSION.toLocaleString()}`);
    console.log(`  Reach: ${post.MEMBERS_REACHED.toLocaleString()}`);
    console.log(`  Engagement: ${post.totalEngagement} (${post.REACTION} reactions, ${post.COMMENT} comments, ${post.RESHARE} reshares)`);
    console.log(`  Engagement Rate: ${((post.totalEngagement / post.IMPRESSION) * 100).toFixed(2)}%`);
  });

  const totalEngagement = totals.REACTION + totals.COMMENT + totals.RESHARE;
  console.log('\n' + '='.repeat(60));
  console.log('TOTALS:');
  console.log(`  Total Impressions: ${totals.IMPRESSION.toLocaleString()}`);
  console.log(`  Total Reach: ${totals.MEMBERS_REACHED.toLocaleString()}`);
  console.log(`  Total Engagement: ${totalEngagement}`);
  console.log(`  Avg Engagement Rate: ${((totalEngagement / totals.IMPRESSION) * 100).toFixed(2)}%`);

  return { postStats, totals };
}

// Usage
const postedIds = [
  'urn:li:share:7123456789012345678',
  'urn:li:share:7234567890123456789',
  'urn:li:share:7345678901234567890'
];

generateReport('linkedin-ABC123DEF', postedIds);
```

### Python Implementation

```python
import requests
from datetime import datetime

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

def get_post_analytics(platform_id, post_urns):
    results = []

    for posted_id in post_urns:
        response = requests.post(
            f'{BASE_URL}/linkedin-post-statistics',
            headers={
                'Content-Type': 'application/json',
                'x-publora-key': PUBLORA_API_KEY
            },
            json={
                'platformId': platform_id,
                'postedId': posted_id,
                'queryTypes': 'ALL'
            }
        )

        data = response.json()
        metrics = data['metrics']
        metrics['postedId'] = posted_id
        metrics['totalEngagement'] = metrics['REACTION'] + metrics['COMMENT'] + metrics['RESHARE']
        results.append(metrics)

    return results

def generate_report(platform_id, post_urns):
    print('=== LinkedIn Analytics Report ===')
    print(f'Generated: {datetime.now().strftime("%Y-%m-%d %H:%M")}\n')

    post_stats = get_post_analytics(platform_id, post_urns)

    totals = {
        'IMPRESSION': sum(p['IMPRESSION'] for p in post_stats),
        'MEMBERS_REACHED': sum(p['MEMBERS_REACHED'] for p in post_stats),
        'REACTION': sum(p['REACTION'] for p in post_stats),
        'COMMENT': sum(p['COMMENT'] for p in post_stats),
        'RESHARE': sum(p['RESHARE'] for p in post_stats)
    }

    print('Post Performance:')
    print('-' * 60)

    for i, post in enumerate(post_stats, 1):
        engagement_rate = (post['totalEngagement'] / post['IMPRESSION'] * 100) if post['IMPRESSION'] > 0 else 0
        print(f"\nPost {i}: {post['postedId']}")
        print(f"  Impressions: {post['IMPRESSION']:,}")
        print(f"  Reach: {post['MEMBERS_REACHED']:,}")
        print(f"  Engagement: {post['totalEngagement']} ({post['REACTION']} reactions, {post['COMMENT']} comments, {post['RESHARE']} reshares)")
        print(f"  Engagement Rate: {engagement_rate:.2f}%")

    total_engagement = totals['REACTION'] + totals['COMMENT'] + totals['RESHARE']
    avg_engagement_rate = (total_engagement / totals['IMPRESSION'] * 100) if totals['IMPRESSION'] > 0 else 0

    print('\n' + '=' * 60)
    print('TOTALS:')
    print(f"  Total Impressions: {totals['IMPRESSION']:,}")
    print(f"  Total Reach: {totals['MEMBERS_REACHED']:,}")
    print(f"  Total Engagement: {total_engagement}")
    print(f"  Avg Engagement Rate: {avg_engagement_rate:.2f}%")

    return {'post_stats': post_stats, 'totals': totals}

# Usage
post_urns = [
    'urn:li:share:7123456789012345678',
    'urn:li:share:7234567890123456789',
    'urn:li:share:7345678901234567890'
]

report = generate_report('linkedin-ABC123DEF', post_urns)
```

## Best Practices

### 1. Wait Before Fetching Analytics

LinkedIn analytics take time to populate. Wait at least 24 hours after publishing for meaningful data.

### 2. Use Caching Wisely

Results are cached for 30 minutes (TOTAL) or 15 minutes (DAILY). The `cached` field in the response tells you if data was served from cache.

### 3. Calculate Engagement Rate

```javascript
function calculateEngagementRate(metrics) {
  const engagement = metrics.REACTION + metrics.COMMENT + metrics.RESHARE;
  return metrics.IMPRESSION > 0
    ? (engagement / metrics.IMPRESSION) * 100
    : 0;
}

// Good engagement rate for LinkedIn: 2-5%
// Great engagement rate: 5%+
```

## Error Handling

```javascript
async function getStatisticsSafe(platformId, postedId) {
  try {
    const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
      },
      body: JSON.stringify({ platformId, postedId, queryTypes: 'ALL' })
    });

    if (!response.ok) {
      const error = await response.json();

      if (response.status === 404) {
        console.log('LinkedIn connection not found');
        return null;
      }

      throw new Error(error.error || 'Failed to fetch statistics');
    }

    return response.json();
  } catch (error) {
    console.error('Analytics error:', error.message);
    return null;
  }
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
