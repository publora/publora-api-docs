# LinkedIn Analytics Guide

Track the performance of your LinkedIn posts with Publora's analytics endpoints.

## Overview

Publora provides analytics for LinkedIn posts, including:
- **Post-level metrics:** impressions, clicks, likes, comments, shares
- **Account-level metrics:** follower growth, engagement rates

> **Note:** Analytics are currently available for LinkedIn only. Other platforms are planned for future releases.

## Available Metrics

### Post Statistics

| Metric | Description |
|--------|-------------|
| `impressions` | Number of times the post was shown |
| `clicks` | Number of clicks on the post |
| `likes` | Number of likes received |
| `comments` | Number of comments |
| `shares` | Number of times the post was shared |

### Account Statistics

| Metric | Description |
|--------|-------------|
| `followers` | Current follower count |
| `followersGrowth` | Net follower change in period |
| `impressions` | Total impressions across all posts |
| `engagementRate` | Average engagement rate |

## Get Post Statistics

### Endpoint

```
POST /api/v1/linkedin-post-statistics
```

### Request

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformConnectionId: 'linkedin-ABC123DEF',
    postUrn: 'urn:li:share:7123456789012345678'
  })
});

const stats = await response.json();
console.log('Impressions:', stats.impressions);
console.log('Engagement:', stats.likes + stats.comments + stats.shares);
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
        'platformConnectionId': 'linkedin-ABC123DEF',
        'postUrn': 'urn:li:share:7123456789012345678'
    }
)

stats = response.json()
print(f"Impressions: {stats['impressions']}")
print(f"Engagement: {stats['likes'] + stats['comments'] + stats['shares']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-post-statistics \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformConnectionId": "linkedin-ABC123DEF",
    "postUrn": "urn:li:share:7123456789012345678"
  }'
```

### Response

```json
{
  "impressions": 1250,
  "clicks": 45,
  "likes": 28,
  "comments": 5,
  "shares": 3,
  "engagementRate": 2.88
}
```

## Get Account Statistics

### Endpoint

```
POST /api/v1/linkedin-account-statistics
```

### Request

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-account-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    platformConnectionId: 'linkedin-ABC123DEF'
  })
});

const accountStats = await response.json();
console.log('Followers:', accountStats.followers);
console.log('Growth:', accountStats.followersGrowth);
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
        'platformConnectionId': 'linkedin-ABC123DEF'
    }
)

account_stats = response.json()
print(f"Followers: {account_stats['followers']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-account-statistics \
  -H "x-publora-key: YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "platformConnectionId": "linkedin-ABC123DEF"
  }'
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

// Find the LinkedIn post URN
const linkedinPost = postDetails.posts.find(p => p.platform === 'linkedin');
const postUrn = linkedinPost.platformPostId; // e.g., "urn:li:share:7123456789012345678"
```

## Analytics Dashboard Example

Build a simple analytics tracker for your LinkedIn posts.

### JavaScript Implementation

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

async function getPostAnalytics(platformConnectionId, postUrns) {
  const results = [];

  for (const postUrn of postUrns) {
    const response = await fetch(`${BASE_URL}/linkedin-post-statistics`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': PUBLORA_API_KEY
      },
      body: JSON.stringify({ platformConnectionId, postUrn })
    });

    const stats = await response.json();
    results.push({
      postUrn,
      ...stats,
      totalEngagement: stats.likes + stats.comments + stats.shares
    });
  }

  return results;
}

async function generateReport(platformConnectionId, postUrns) {
  console.log('=== LinkedIn Analytics Report ===\n');

  // Get post stats
  const postStats = await getPostAnalytics(platformConnectionId, postUrns);

  // Calculate totals
  const totals = postStats.reduce((acc, post) => ({
    impressions: acc.impressions + post.impressions,
    clicks: acc.clicks + post.clicks,
    likes: acc.likes + post.likes,
    comments: acc.comments + post.comments,
    shares: acc.shares + post.shares
  }), { impressions: 0, clicks: 0, likes: 0, comments: 0, shares: 0 });

  // Print per-post stats
  console.log('Post Performance:');
  console.log('-'.repeat(60));

  postStats.forEach((post, i) => {
    console.log(`\nPost ${i + 1}: ${post.postUrn}`);
    console.log(`  Impressions: ${post.impressions.toLocaleString()}`);
    console.log(`  Engagement: ${post.totalEngagement} (${post.likes} likes, ${post.comments} comments, ${post.shares} shares)`);
    console.log(`  Click-through: ${post.clicks} clicks`);
    console.log(`  Engagement Rate: ${((post.totalEngagement / post.impressions) * 100).toFixed(2)}%`);
  });

  // Print totals
  console.log('\n' + '='.repeat(60));
  console.log('TOTALS:');
  console.log(`  Total Impressions: ${totals.impressions.toLocaleString()}`);
  console.log(`  Total Engagement: ${totals.likes + totals.comments + totals.shares}`);
  console.log(`  Total Clicks: ${totals.clicks}`);
  console.log(`  Avg Engagement Rate: ${(((totals.likes + totals.comments + totals.shares) / totals.impressions) * 100).toFixed(2)}%`);

  return { postStats, totals };
}

// Usage
const postUrns = [
  'urn:li:share:7123456789012345678',
  'urn:li:share:7234567890123456789',
  'urn:li:share:7345678901234567890'
];

generateReport('linkedin-ABC123DEF', postUrns);
```

### Python Implementation

```python
import requests
from datetime import datetime

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

def get_post_analytics(platform_connection_id, post_urns):
    results = []

    for post_urn in post_urns:
        response = requests.post(
            f'{BASE_URL}/linkedin-post-statistics',
            headers={
                'Content-Type': 'application/json',
                'x-publora-key': PUBLORA_API_KEY
            },
            json={
                'platformConnectionId': platform_connection_id,
                'postUrn': post_urn
            }
        )

        stats = response.json()
        stats['postUrn'] = post_urn
        stats['totalEngagement'] = stats['likes'] + stats['comments'] + stats['shares']
        results.append(stats)

    return results

def generate_report(platform_connection_id, post_urns):
    print('=== LinkedIn Analytics Report ===')
    print(f'Generated: {datetime.now().strftime("%Y-%m-%d %H:%M")}\n')

    # Get post stats
    post_stats = get_post_analytics(platform_connection_id, post_urns)

    # Calculate totals
    totals = {
        'impressions': sum(p['impressions'] for p in post_stats),
        'clicks': sum(p['clicks'] for p in post_stats),
        'likes': sum(p['likes'] for p in post_stats),
        'comments': sum(p['comments'] for p in post_stats),
        'shares': sum(p['shares'] for p in post_stats)
    }

    # Print per-post stats
    print('Post Performance:')
    print('-' * 60)

    for i, post in enumerate(post_stats, 1):
        engagement_rate = (post['totalEngagement'] / post['impressions'] * 100) if post['impressions'] > 0 else 0
        print(f"\nPost {i}: {post['postUrn']}")
        print(f"  Impressions: {post['impressions']:,}")
        print(f"  Engagement: {post['totalEngagement']} ({post['likes']} likes, {post['comments']} comments, {post['shares']} shares)")
        print(f"  Click-through: {post['clicks']} clicks")
        print(f"  Engagement Rate: {engagement_rate:.2f}%")

    # Print totals
    total_engagement = totals['likes'] + totals['comments'] + totals['shares']
    avg_engagement_rate = (total_engagement / totals['impressions'] * 100) if totals['impressions'] > 0 else 0

    print('\n' + '=' * 60)
    print('TOTALS:')
    print(f"  Total Impressions: {totals['impressions']:,}")
    print(f"  Total Engagement: {total_engagement}")
    print(f"  Total Clicks: {totals['clicks']}")
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

```javascript
// Don't do this immediately after posting
const stats = await getPostStatistics(postUrn); // Will show zeros

// Instead, schedule analytics collection for the next day
```

### 2. Track Trends Over Time

Store analytics data to track performance trends:

```javascript
const analyticsHistory = [];

async function trackDaily(platformConnectionId, postUrn) {
  const stats = await getPostStatistics(platformConnectionId, postUrn);

  analyticsHistory.push({
    date: new Date().toISOString().split('T')[0],
    postUrn,
    ...stats
  });

  // Calculate daily change
  if (analyticsHistory.length > 1) {
    const yesterday = analyticsHistory[analyticsHistory.length - 2];
    console.log(`New impressions today: ${stats.impressions - yesterday.impressions}`);
  }
}
```

### 3. Calculate Engagement Rate

```javascript
function calculateEngagementRate(stats) {
  const engagement = stats.likes + stats.comments + stats.shares;
  return stats.impressions > 0
    ? (engagement / stats.impressions) * 100
    : 0;
}

// Good engagement rate for LinkedIn: 2-5%
// Great engagement rate: 5%+
```

### 4. Compare Post Performance

```javascript
function findTopPerformer(postStats) {
  return postStats.reduce((best, current) => {
    const currentRate = calculateEngagementRate(current);
    const bestRate = calculateEngagementRate(best);
    return currentRate > bestRate ? current : best;
  });
}
```

## Error Handling

```javascript
async function getStatisticsSafe(platformConnectionId, postUrn) {
  try {
    const response = await fetch(`${BASE_URL}/linkedin-post-statistics`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': PUBLORA_API_KEY
      },
      body: JSON.stringify({ platformConnectionId, postUrn })
    });

    if (!response.ok) {
      const error = await response.json();

      if (response.status === 404) {
        console.log('Post not found or not yet published');
        return null;
      }

      throw new Error(error.message || 'Failed to fetch statistics');
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
