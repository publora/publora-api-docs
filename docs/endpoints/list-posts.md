# List Posts

Retrieve a paginated list of all your scheduled, draft, published, and failed posts.

## Endpoint

```
GET https://api.publora.com/api/v1/list-posts
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |

## Query Parameters

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `page` | number | `1` | Page number (1-indexed). Values less than 1 are silently clamped to 1. |
| `limit` | number | `20` | Items per page. Values are silently clamped to the range 1-100. |
| `status` | string | all | Filter by status: `draft`, `scheduled`, `published`, `failed`, `partially_published` |
| `platform` | string | all | Filter by platform (case-insensitive prefix matching): `twitter`, `linkedin`, `instagram`, `threads`, `tiktok`, `youtube`, `facebook`, `bluesky`, `mastodon`, `telegram` |
| `sortBy` | string | `createdAt` | Sort field: `createdAt`, `scheduledTime`, `updatedAt` |
| `sortOrder` | string | `desc` | Sort order: `asc` or `desc` |
| `fromDate` | string | - | Filter posts scheduled after this ISO 8601 date |
| `toDate` | string | - | Filter posts scheduled before this ISO 8601 date |

## Response

```json
{
  "success": true,
  "posts": [
    {
      "postGroupId": "507f1f77bcf86cd799439011",
      "content": "Excited to share our new product launch!",
      "status": "scheduled",
      "scheduledTime": "2026-03-15T14:30:00.000Z",
      "createdAt": "2026-03-10T09:00:00.000Z",
      "updatedAt": "2026-03-10T09:00:00.000Z",
      "platforms": [
        {
          "platformId": "twitter-123456789",
          "platform": "twitter",
          "status": "scheduled"
        },
        {
          "platformId": "linkedin-ABC123",
          "platform": "linkedin",
          "status": "scheduled"
        }
      ],
      "mediaUrls": ["https://cdn.publora.com/media/abc123.jpg"]
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "totalItems": 47,
    "totalPages": 3,
    "hasNextPage": true,
    "hasPrevPage": false
  }
}
```

## Pagination Fields

| Field | Type | Description |
|-------|------|-------------|
| `page` | number | Current page number |
| `limit` | number | Items per page |
| `totalItems` | number | Total number of posts matching filters |
| `totalPages` | number | Total number of pages |
| `hasNextPage` | boolean | Whether more pages exist after current |
| `hasPrevPage` | boolean | Whether pages exist before current |

## Examples

### JavaScript - Fetch All Scheduled Posts with Pagination

```javascript
async function fetchAllScheduledPosts(apiKey) {
  const posts = [];
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const url = new URL('https://api.publora.com/api/v1/list-posts');
    url.searchParams.set('page', page);
    url.searchParams.set('limit', 100);
    url.searchParams.set('status', 'scheduled');
    url.searchParams.set('sortBy', 'scheduledTime');
    url.searchParams.set('sortOrder', 'asc');

    const response = await fetch(url, {
      headers: { 'x-publora-key': apiKey }
    });
    const data = await response.json();

    posts.push(...data.posts);
    hasMore = data.pagination.hasNextPage;
    page++;

    console.log(`Fetched page ${data.pagination.page} of ${data.pagination.totalPages}`);
  }

  console.log(`Total scheduled posts: ${posts.length}`);
  return posts;
}

// Usage
const scheduledPosts = await fetchAllScheduledPosts('YOUR_API_KEY');
```

### JavaScript - Paginated List with UI Controls

```javascript
class PostListPaginator {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.publora.com/api/v1/list-posts';
    this.currentPage = 1;
    this.limit = 20;
    this.filters = {};
  }

  setFilters({ status, platform, fromDate, toDate }) {
    this.filters = { status, platform, fromDate, toDate };
    this.currentPage = 1; // Reset to first page when filters change
  }

  async fetchPage(page = this.currentPage) {
    const url = new URL(this.baseUrl);
    url.searchParams.set('page', page);
    url.searchParams.set('limit', this.limit);

    // Apply filters
    if (this.filters.status) url.searchParams.set('status', this.filters.status);
    if (this.filters.platform) url.searchParams.set('platform', this.filters.platform);
    if (this.filters.fromDate) url.searchParams.set('fromDate', this.filters.fromDate);
    if (this.filters.toDate) url.searchParams.set('toDate', this.filters.toDate);

    const response = await fetch(url, {
      headers: { 'x-publora-key': this.apiKey }
    });
    const data = await response.json();

    this.currentPage = data.pagination.page;
    return data;
  }

  async nextPage() {
    return this.fetchPage(this.currentPage + 1);
  }

  async prevPage() {
    return this.fetchPage(Math.max(1, this.currentPage - 1));
  }

  async goToPage(page) {
    return this.fetchPage(page);
  }
}

// Usage
const paginator = new PostListPaginator('YOUR_API_KEY');

// Get first page of scheduled posts
paginator.setFilters({ status: 'scheduled' });
const firstPage = await paginator.fetchPage();

console.log(`Showing ${firstPage.posts.length} of ${firstPage.pagination.totalItems} posts`);

// Navigate pages
if (firstPage.pagination.hasNextPage) {
  const secondPage = await paginator.nextPage();
}
```

### Python - Fetch All Posts with Pagination

```python
import requests
from typing import Generator, Dict, Any, Optional

def fetch_posts_paginated(
    api_key: str,
    status: Optional[str] = None,
    platform: Optional[str] = None,
    limit: int = 100
) -> Generator[Dict[str, Any], None, None]:
    """
    Generator that yields posts one by one, handling pagination automatically.

    Args:
        api_key: Your Publora API key
        status: Filter by status (draft, scheduled, published, failed)
        platform: Filter by platform (twitter, linkedin, etc.)
        limit: Items per page (max 100)

    Yields:
        Individual post objects
    """
    base_url = 'https://api.publora.com/api/v1/list-posts'
    headers = {'x-publora-key': api_key}
    page = 1

    while True:
        params = {
            'page': page,
            'limit': limit,
            'sortBy': 'scheduledTime',
            'sortOrder': 'asc'
        }

        if status:
            params['status'] = status
        if platform:
            params['platform'] = platform

        response = requests.get(base_url, headers=headers, params=params)
        response.raise_for_status()
        data = response.json()

        for post in data['posts']:
            yield post

        if not data['pagination']['hasNextPage']:
            break

        page += 1


def fetch_all_scheduled_posts(api_key: str) -> list:
    """Fetch all scheduled posts into a list."""
    posts = list(fetch_posts_paginated(api_key, status='scheduled'))
    print(f"Fetched {len(posts)} scheduled posts")
    return posts


# Usage
api_key = 'YOUR_API_KEY'

# Iterate through all scheduled posts
for post in fetch_posts_paginated(api_key, status='scheduled'):
    print(f"{post['postGroupId']}: {post['content'][:50]}...")
    print(f"  Scheduled for: {post['scheduledTime']}")
    print(f"  Platforms: {[p['platform'] for p in post['platforms']]}")

# Or fetch all at once
all_posts = fetch_all_scheduled_posts(api_key)
```

### Python - Filter Posts by Date Range

```python
import requests
from datetime import datetime, timedelta, timezone

def get_posts_for_week(api_key: str, start_date: datetime) -> list:
    """Get all posts scheduled for a specific week."""
    end_date = start_date + timedelta(days=7)

    params = {
        'status': 'scheduled',
        'fromDate': start_date.isoformat(),
        'toDate': end_date.isoformat(),
        'sortBy': 'scheduledTime',
        'sortOrder': 'asc',
        'limit': 100
    }

    response = requests.get(
        'https://api.publora.com/api/v1/list-posts',
        headers={'x-publora-key': api_key},
        params=params
    )

    return response.json()['posts']


# Usage: Get posts scheduled for next week
next_monday = datetime.now(timezone.utc).replace(
    hour=0, minute=0, second=0, microsecond=0
)
# Adjust to next Monday
days_until_monday = (7 - next_monday.weekday()) % 7
next_monday += timedelta(days=days_until_monday)

posts = get_posts_for_week('YOUR_API_KEY', next_monday)
print(f"Posts scheduled for next week: {len(posts)}")
```

### Node.js (axios) - Paginated Fetch with Async Iterator

```javascript
const axios = require('axios');

async function* paginatedPosts(apiKey, options = {}) {
  const { status, platform, limit = 100 } = options;
  let page = 1;
  let hasMore = true;

  const api = axios.create({
    baseURL: 'https://api.publora.com/api/v1',
    headers: { 'x-publora-key': apiKey }
  });

  while (hasMore) {
    const params = { page, limit, sortBy: 'scheduledTime', sortOrder: 'asc' };
    if (status) params.status = status;
    if (platform) params.platform = platform;

    const { data } = await api.get('/list-posts', { params });

    for (const post of data.posts) {
      yield post;
    }

    hasMore = data.pagination.hasNextPage;
    page++;
  }
}

// Usage
(async () => {
  // Iterate through all scheduled posts
  for await (const post of paginatedPosts('YOUR_API_KEY', { status: 'scheduled' })) {
    console.log(`${post.postGroupId}: ${post.content.slice(0, 50)}...`);
    console.log(`  Scheduled: ${post.scheduledTime}`);
  }
})();
```

### cURL - Basic Pagination

```bash
# Get first page of scheduled posts
curl "https://api.publora.com/api/v1/list-posts?page=1&limit=20&status=scheduled" \
  -H "x-publora-key: YOUR_API_KEY"

# Get second page
curl "https://api.publora.com/api/v1/list-posts?page=2&limit=20&status=scheduled" \
  -H "x-publora-key: YOUR_API_KEY"

# Filter by platform and date range
curl "https://api.publora.com/api/v1/list-posts?status=scheduled&platform=twitter&fromDate=2026-03-01T00:00:00Z&toDate=2026-03-31T23:59:59Z" \
  -H "x-publora-key: YOUR_API_KEY"

# Get all posts sorted by scheduled time
curl "https://api.publora.com/api/v1/list-posts?sortBy=scheduledTime&sortOrder=asc&limit=100" \
  -H "x-publora-key: YOUR_API_KEY"
```

### Bash - Fetch All Pages

```bash
#!/bin/bash

API_KEY="YOUR_API_KEY"
BASE_URL="https://api.publora.com/api/v1/list-posts"
PAGE=1
LIMIT=100
ALL_POSTS="[]"

while true; do
  RESPONSE=$(curl -s "${BASE_URL}?page=${PAGE}&limit=${LIMIT}&status=scheduled" \
    -H "x-publora-key: ${API_KEY}")

  POSTS=$(echo "$RESPONSE" | jq '.posts')
  HAS_NEXT=$(echo "$RESPONSE" | jq '.pagination.hasNextPage')
  TOTAL=$(echo "$RESPONSE" | jq '.pagination.totalItems')

  # Merge posts
  ALL_POSTS=$(echo "$ALL_POSTS" "$POSTS" | jq -s 'add')

  echo "Fetched page $PAGE (Total items: $TOTAL)"

  if [ "$HAS_NEXT" = "false" ]; then
    break
  fi

  PAGE=$((PAGE + 1))
done

echo "Total posts fetched: $(echo "$ALL_POSTS" | jq 'length')"
echo "$ALL_POSTS" > scheduled_posts.json
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Invalid status. Must be one of: draft, scheduled, published, failed, partially_published"` | Invalid status filter value |
| 400 | `"Invalid sortBy. Must be one of: createdAt, updatedAt, scheduledTime"` | Invalid sortBy field |
| 400 | `"Invalid fromDate format"` | fromDate is not a valid ISO 8601 date |
| 400 | `"Invalid toDate format"` | toDate is not a valid ISO 8601 date |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |

> **Note:** The `page` and `limit` parameters do not return errors for out-of-range values. Instead, they are silently clamped to valid ranges: `page` is clamped to a minimum of 1, and `limit` is clamped to the range 1-100.

## Best Practices

1. **Use reasonable page sizes.** For UI pagination, 20-50 items is typical. For bulk exports, use the max of 100.

2. **Always handle pagination.** Don't assume all posts fit on one page. Check `hasNextPage` or loop until `page >= totalPages`.

3. **Cache total counts.** The `totalItems` count is useful for UI but may be expensive. Cache it for paginated UIs.

4. **Filter server-side.** Use query parameters to filter by status and platform rather than fetching all posts and filtering client-side.

5. **Use date ranges for calendars.** When building calendar views, use `fromDate` and `toDate` to fetch only the visible range.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
