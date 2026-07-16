# How to Retrieve LinkedIn Analytics with Publora

Get engagement metrics (impressions, reactions, comments, shares) for your LinkedIn posts using the Publora API.

## Overview

Publora provides analytics for LinkedIn posts through a single endpoint. You can retrieve individual metrics or all metrics at once.

## API Reference

**Endpoint:**
```
POST https://api.publora.com/api/v1/linkedin-post-statistics
```

**Headers:**
```
Content-Type: application/json
x-publora-key: YOUR_API_KEY
```

**Request Body:**
```json
{
  "postedId": "urn:li:share:7434685316856377344",
  "platformId": "linkedin-ABC123",
  "queryTypes": ["ALL"]
}
```

## Available Metrics

| Metric | Description |
|--------|-------------|
| `IMPRESSION` | Number of times the post was displayed |
| `MEMBERS_REACHED` | Unique LinkedIn members who saw the post |
| `REACTION` | Total reactions (likes, celebrates, etc.) |
| `COMMENT` | Number of comments on the post |
| `RESHARE` | Number of times the post was shared |

## Request Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN (from get-post endpoint) |
| `platformId` | string | Yes | Your LinkedIn platform ID (e.g., `linkedin-ABC123`) |
| `queryType` | string | No | Single metric: `IMPRESSION`, `REACTION`, etc. |
| `queryTypes` | array | No | Multiple metrics: `["IMPRESSION", "REACTION"]` or `["ALL"]` |

*Use either `queryType` (single) or `queryTypes` (multiple), not both.*

## Response Format

**Single metric response:**
```json
{
  "success": true,
  "count": 4521,
  "cached": false
}
```

**Multiple metrics response:**
```json
{
  "success": true,
  "metrics": {
    "IMPRESSION": 4521,
    "MEMBERS_REACHED": 3200,
    "REACTION": 89,
    "COMMENT": 15,
    "RESHARE": 12
  },
  "cached": false,
  "cachedMetrics": []
}
```

## Complete JavaScript Implementation

```javascript
const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY;
const BASE_URL = 'https://api.publora.com/api/v1';

/**
 * Retrieve LinkedIn post analytics
 * @param {string} postedId - LinkedIn post URN (urn:li:share:xxx or urn:li:ugcPost:xxx)
 * @param {string} platformId - LinkedIn platform ID (linkedin-xxx)
 * @param {string|string[]} metrics - Single metric, array of metrics, or 'ALL'
 * @returns {Promise<Object>} Analytics data
 */
async function getLinkedInAnalytics(postedId, platformId, metrics = 'ALL') {
  // Validate inputs
  if (!postedId) {
    throw new Error('postedId is required');
  }
  if (!postedId.startsWith('urn:li:')) {
    throw new Error('postedId must be a LinkedIn URN (urn:li:share:xxx or urn:li:ugcPost:xxx)');
  }
  if (!platformId) {
    throw new Error('platformId is required');
  }

  // Build request body
  const body = {
    postedId,
    platformId
  };

  // Handle metrics parameter
  if (typeof metrics === 'string') {
    if (metrics === 'ALL') {
      body.queryTypes = ['ALL'];
    } else {
      body.queryType = metrics;
    }
  } else if (Array.isArray(metrics)) {
    body.queryTypes = metrics;
  }

  // Make API request
  const response = await fetch(`${BASE_URL}/linkedin-post-statistics`, {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': PUBLORA_API_KEY
    },
    body: JSON.stringify(body)
  });

  const data = await response.json();

  // Handle errors
  if (!response.ok) {
    const error = new Error(data.error || `HTTP ${response.status}`);
    error.status = response.status;
    error.response = data;

    switch (response.status) {
      case 400:
        error.message = `Invalid request: ${data.error}`;
        break;
      case 401:
        error.message = 'Invalid API key. Check your x-publora-key header.';
        break;
      case 404:
        error.message = 'LinkedIn connection not found. Verify platformId.';
        break;
      case 500:
        error.message = 'Failed to fetch analytics. The post may not exist or analytics are not yet available.';
        break;
    }

    throw error;
  }

  return data;
}

// ============ USAGE EXAMPLES ============

// Example 1: Get all metrics
async function getAllMetrics(postedId, platformId) {
  try {
    const analytics = await getLinkedInAnalytics(postedId, platformId, 'ALL');
    console.log('Impressions:', analytics.metrics.IMPRESSION);
    console.log('Reactions:', analytics.metrics.REACTION);
    console.log('Comments:', analytics.metrics.COMMENT);
    console.log('Shares:', analytics.metrics.RESHARE);
    console.log('Reach:', analytics.metrics.MEMBERS_REACHED);
    return analytics;
  } catch (error) {
    console.error('Failed to get analytics:', error.message);
    throw error;
  }
}

// Example 2: Get specific metrics only
async function getEngagementMetrics(postedId, platformId) {
  try {
    const analytics = await getLinkedInAnalytics(
      postedId,
      platformId,
      ['REACTION', 'COMMENT', 'RESHARE']
    );
    return analytics.metrics;
  } catch (error) {
    console.error('Failed to get engagement:', error.message);
    throw error;
  }
}

// Example 3: Get single metric
async function getImpressions(postedId, platformId) {
  try {
    const analytics = await getLinkedInAnalytics(postedId, platformId, 'IMPRESSION');
    return analytics.count;
  } catch (error) {
    console.error('Failed to get impressions:', error.message);
    throw error;
  }
}

// Example 4: Get analytics for a post created via Publora
async function getAnalyticsForPost(postGroupId, platformId) {
  // Step 1: Get the postedId from the post
  const postResponse = await fetch(`${BASE_URL}/get-post/${postGroupId}`, {
    headers: { 'x-publora-key': PUBLORA_API_KEY }
  });
  const postData = await postResponse.json();

  const linkedInPost = postData.posts.find(p => p.platform === 'linkedin');
  if (!linkedInPost || !linkedInPost.postedId) {
    throw new Error('LinkedIn post not found or not yet published');
  }

  // Step 2: Get analytics
  return await getLinkedInAnalytics(linkedInPost.postedId, platformId, 'ALL');
}

// Run examples
const postedId = 'urn:li:share:7434685316856377344';
const platformId = 'linkedin-ABC123';

const allMetrics = await getAllMetrics(postedId, platformId);
const engagement = await getEngagementMetrics(postedId, platformId);
const impressions = await getImpressions(postedId, platformId);
```

## Complete Python Implementation

```python
import os
import requests
from typing import Union, List, Dict, Optional

PUBLORA_API_KEY = os.environ.get('PUBLORA_API_KEY')
BASE_URL = 'https://api.publora.com/api/v1'


class PubloraAnalyticsError(Exception):
    """Custom exception for analytics API errors."""
    def __init__(self, message: str, status_code: int = None, response: dict = None):
        super().__init__(message)
        self.status_code = status_code
        self.response = response


def get_linkedin_analytics(
    posted_id: str,
    platform_id: str,
    metrics: Union[str, List[str]] = 'ALL'
) -> Dict:
    """
    Retrieve LinkedIn post analytics.

    Args:
        posted_id: LinkedIn post URN (urn:li:share:xxx or urn:li:ugcPost:xxx)
        platform_id: LinkedIn platform ID (linkedin-xxx)
        metrics: Single metric string, list of metrics, or 'ALL'

    Returns:
        dict: Analytics data with metrics

    Raises:
        PubloraAnalyticsError: If the API request fails
        ValueError: If inputs are invalid
    """
    # Validate inputs
    if not posted_id:
        raise ValueError('posted_id is required')
    if not posted_id.startswith('urn:li:'):
        raise ValueError('posted_id must be a LinkedIn URN (urn:li:share:xxx or urn:li:ugcPost:xxx)')
    if not platform_id:
        raise ValueError('platform_id is required')

    # Build request body
    body = {
        'postedId': posted_id,
        'platformId': platform_id
    }

    # Handle metrics parameter
    if isinstance(metrics, str):
        if metrics == 'ALL':
            body['queryTypes'] = ['ALL']
        else:
            body['queryType'] = metrics
    elif isinstance(metrics, list):
        body['queryTypes'] = metrics

    # Make API request
    response = requests.post(
        f'{BASE_URL}/linkedin-post-statistics',
        headers={
            'Content-Type': 'application/json',
            'x-publora-key': PUBLORA_API_KEY
        },
        json=body
    )

    data = response.json()

    # Handle errors
    if not response.ok:
        error_message = data.get('error', f'HTTP {response.status_code}')

        if response.status_code == 400:
            error_message = f'Invalid request: {error_message}'
        elif response.status_code == 401:
            error_message = 'Invalid API key. Check your x-publora-key header.'
        elif response.status_code == 404:
            error_message = 'LinkedIn connection not found. Verify platform_id.'
        elif response.status_code == 500:
            error_message = 'Failed to fetch analytics. The post may not exist or analytics are not yet available.'

        raise PubloraAnalyticsError(error_message, response.status_code, data)

    return data


# ============ USAGE EXAMPLES ============

def get_all_metrics(posted_id: str, platform_id: str) -> Dict:
    """Get all available metrics for a post."""
    try:
        analytics = get_linkedin_analytics(posted_id, platform_id, 'ALL')
        print(f"Impressions: {analytics['metrics']['IMPRESSION']}")
        print(f"Reactions: {analytics['metrics']['REACTION']}")
        print(f"Comments: {analytics['metrics']['COMMENT']}")
        print(f"Shares: {analytics['metrics']['RESHARE']}")
        print(f"Reach: {analytics['metrics']['MEMBERS_REACHED']}")
        return analytics
    except (PubloraAnalyticsError, ValueError) as e:
        print(f"Failed to get analytics: {e}")
        raise


def get_engagement_metrics(posted_id: str, platform_id: str) -> Dict:
    """Get engagement metrics only (reactions, comments, shares)."""
    try:
        analytics = get_linkedin_analytics(
            posted_id,
            platform_id,
            ['REACTION', 'COMMENT', 'RESHARE']
        )
        return analytics['metrics']
    except (PubloraAnalyticsError, ValueError) as e:
        print(f"Failed to get engagement: {e}")
        raise


def get_impressions(posted_id: str, platform_id: str) -> int:
    """Get impression count for a post."""
    try:
        analytics = get_linkedin_analytics(posted_id, platform_id, 'IMPRESSION')
        return analytics['count']
    except (PubloraAnalyticsError, ValueError) as e:
        print(f"Failed to get impressions: {e}")
        raise


def get_analytics_for_post(post_group_id: str, platform_id: str) -> Dict:
    """Get analytics for a post created via Publora."""
    # Step 1: Get the posted_id from the post
    post_response = requests.get(
        f'{BASE_URL}/get-post/{post_group_id}',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )
    post_data = post_response.json()

    linkedin_post = next(
        (p for p in post_data.get('posts', []) if p.get('platform') == 'linkedin'),
        None
    )

    if not linkedin_post or not linkedin_post.get('postedId'):
        raise ValueError('LinkedIn post not found or not yet published')

    # Step 2: Get analytics
    return get_linkedin_analytics(linkedin_post['postedId'], platform_id, 'ALL')


# Run examples
if __name__ == '__main__':
    posted_id = 'urn:li:share:7434685316856377344'
    platform_id = 'linkedin-ABC123'

    all_metrics = get_all_metrics(posted_id, platform_id)
    engagement = get_engagement_metrics(posted_id, platform_id)
    impressions = get_impressions(posted_id, platform_id)
```

## cURL Examples

### Get all metrics

```bash
curl -X POST "https://api.publora.com/api/v1/linkedin-post-statistics" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7434685316856377344",
    "platformId": "linkedin-ABC123",
    "queryTypes": ["ALL"]
  }'
```

### Get specific metrics

```bash
curl -X POST "https://api.publora.com/api/v1/linkedin-post-statistics" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7434685316856377344",
    "platformId": "linkedin-ABC123",
    "queryTypes": ["IMPRESSION", "REACTION", "COMMENT"]
  }'
```

### Get single metric

```bash
curl -X POST "https://api.publora.com/api/v1/linkedin-post-statistics" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7434685316856377344",
    "platformId": "linkedin-ABC123",
    "queryType": "IMPRESSION"
  }'
```

## Important Notes

### Analytics Delay

LinkedIn analytics may take **up to 24 hours** to fully populate. Querying immediately after posting will return partial or zero data.

### Caching

Publora caches post analytics for 2 hours to reduce LinkedIn API calls. The `cached` field indicates if data was served from cache.

### Finding the postedId

For posts created via Publora, get the `postedId` from the get-post endpoint:

```javascript
const response = await fetch(`https://api.publora.com/api/v1/get-post/${postGroupId}`, {
  headers: { 'x-publora-key': 'YOUR_API_KEY' }
});
const data = await response.json();
const postedId = data.posts.find(p => p.platform === 'linkedin')?.postedId;
```

### Rate Limiting

- Analytics requests are subject to LinkedIn's rate limits
- Publora's caching helps reduce rate limit issues
- For bulk analytics, add delays between requests

## Error Reference

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 400 | `"postedId and platformId are required"` | Missing required fields | Include both fields |
| 400 | `"Invalid queryType"` | Unknown metric name | Use valid metric names |
| 401 | `"Invalid API key"` | Bad x-publora-key | Check API key |
| 404 | `"LinkedIn connection not found"` | Wrong platformId | Verify platformId |
| 500 | `"Failed to fetch LinkedIn post statistics"` | LinkedIn API error | Post may not exist or no data yet |

## Related Guides

- [LinkedIn Account Statistics](../endpoints/linkedin-statistics.md) - Account-level analytics
- [LinkedIn Followers](../endpoints/linkedin-followers.md) - Follower count and growth
- [LinkedIn Profile Summary](../endpoints/linkedin-profile-summary.md) - Combined profile stats
- [Get Post](../endpoints/get-post.md) - Get postedId for analytics
