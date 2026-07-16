# Rate Limits and Optimal Posting Times

This guide covers platform-specific rate limits, Publora API rate limits, and strategies for scheduling posts at optimal engagement times.

## Quick Start: High-Volume Multi-Platform Scheduling

For scheduling hundreds of posts across all 10 platforms simultaneously, use this production-ready approach:

### Installation

```bash
# JavaScript/Node.js
npm install axios

# Python
pip install requests
```

### Environment Setup

```bash
# Set your API key (get it from publora.com → API in sidebar)
export PUBLORA_API_KEY="sk_YOUR_API_KEY"
```

### Minimal Working Example

**JavaScript (Node.js):**

```javascript
// high-volume-scheduler.js
// Run: PUBLORA_API_KEY=sk_YOUR_API_KEY node high-volume-scheduler.js

const axios = require('axios');

const API_KEY = process.env.PUBLORA_API_KEY;
const BASE_URL = 'https://api.publora.com/api/v1';

// Optional user-supplied throttles. Publora defines no per-platform quotas.
const PLATFORM_LIMITS = {};

async function schedulePost(content, platforms, scheduledTime) {
  const response = await axios.post(`${BASE_URL}/create-post`, {
    content,
    platforms,
    scheduledTime
  }, {
    headers: {
      'x-publora-key': API_KEY,
      'Content-Type': 'application/json'
    }
  });
  return response.data;
}

async function main() {
  // Your platform connection IDs (from /platform-connections endpoint)
  const platforms = ['twitter-123456', 'linkedin-ABC123', 'threads-789012'];

  // Schedule 10 posts, 1 hour apart
  const now = new Date();
  for (let i = 0; i < 10; i++) {
    const scheduledTime = new Date(now.getTime() + (i + 1) * 3600000);
    const result = await schedulePost(
      `Post ${i + 1}: Your content here`,
      platforms,
      scheduledTime.toISOString()
    );
    console.log(`Scheduled post ${i + 1}: ${result.postGroupId}`);
  }
}

main().catch(console.error);
```

**Python:**

```python
#!/usr/bin/env python3
# high_volume_scheduler.py
# Run: PUBLORA_API_KEY=sk_YOUR_API_KEY python high_volume_scheduler.py

import os
import requests
from datetime import datetime, timedelta, timezone

API_KEY = os.environ['PUBLORA_API_KEY']
BASE_URL = 'https://api.publora.com/api/v1'

# Optional user-supplied throttles. Publora defines no per-platform quotas.
PLATFORM_LIMITS = {}

def schedule_post(content: str, platforms: list, scheduled_time: str) -> dict:
    response = requests.post(
        f'{BASE_URL}/create-post',
        json={
            'content': content,
            'platforms': platforms,
            'scheduledTime': scheduled_time
        },
        headers={
            'x-publora-key': API_KEY,
            'Content-Type': 'application/json'
        }
    )
    response.raise_for_status()
    return response.json()

def main():
    # Your platform connection IDs (from /platform-connections endpoint)
    platforms = ['twitter-123456', 'linkedin-ABC123', 'threads-789012']

    # Schedule 10 posts, 1 hour apart
    now = datetime.now(timezone.utc)
    for i in range(10):
        scheduled_time = now + timedelta(hours=i + 1)
        result = schedule_post(
            f'Post {i + 1}: Your content here',
            platforms,
            scheduled_time.isoformat()
        )
        print(f"Scheduled post {i + 1}: {result['postGroupId']}")

if __name__ == '__main__':
    main()
```

---

## Publora Plan Limits

Each Publora plan enforces limits on monthly posts, scheduled (pending) posts, and how far in advance you can schedule.

| Plan | Monthly Posts | Connections | Post Limit Scope | Scheduled Posts | Schedule Horizon | API/MCP Access | Platforms |
|------|-------------|-------------|------------------|-----------------|------------------|----------------|-----------|
| **Starter (Free)** | 15 | 3 | Account-wide | 3 pending | 7 days | Yes | All 10 |
| **Pro Monthly ($5.99/mo)** | 100 | Per seat* | Per connection | 100 pending | Unlimited | Yes | All |
| **Pro Yearly ($2.99/mo)** | 100 | Per seat* | Per connection | 100 pending | Unlimited | Yes | All |
| **Premium Monthly ($9.99/mo)** | 500 | Per seat* | Per connection | 500 pending | Unlimited | Yes | All |
| **Premium Yearly ($5.99/mo)** | 500 | Per seat* | Per connection | 500 pending | Unlimited | Yes | All |

*\*Paid plan connection limits are based on subscription quantity (seat count), not truly unlimited. Each seat in your subscription adds to your total allowed connections.*

**Key details:**

- **Monthly-post scope:** Starter's 15 monthly posts are account-wide; paid-plan `monthlyPosts` are counted per connection (for example, 100 for one LinkedIn connection and 100 for one Twitter connection). The active `scheduledPosts` queue limit is workspace-wide on every plan, not per connection. Connection count and schedule horizon are separate plan limits.
- **Starter plan includes API and MCP access** (`apiAccess: true`, `mcpAccess: true`) — the free tier can use the REST API and MCP server (3 connected accounts, 15 posts/month account-wide).
- **Starter plan can post to any of the 10 platforms** (no platform restriction).

> **Note:** Per-request API rate limiting (requests/minute) is **planned but not currently enforced**. The limits above (monthly posts, scheduled posts, schedule horizon) are the enforced plan-based limits. Future API rate limiting will be documented here when implemented.

## Platform-Specific Rate Limits (Advisory)

Platform-side posting limits are advisory, account-dependent, and may change without notice. Publora does not treat unsourced posts-per-hour/day figures as an API contract; consult the destination platform for current quotas.

### What Publora enforces

- Plan entitlements shown above (monthly posts, pending posts, connections, and scheduling horizon).
- Media-URL ingestion: 60 URLs per fixed one-hour window; `429 MEDIA_URL_RATE_LIMITED` includes `Retry-After`.
- MCP session limits documented by the MCP endpoint.

Platform-side rate-limit failures are reported on the post. Do not assume Publora automatically retries or redistributes a failed platform publication.

## API Reference for Scheduling

The scheduler examples use these Publora API endpoints:

### Create Post

```http
POST https://api.publora.com/api/v1/create-post
Content-Type: application/json
x-publora-key: sk_YOUR_API_KEY

{
  "content": "Your post content",
  "platforms": ["twitter-123456", "linkedin-ABC123"],
  "scheduledTime": "<FUTURE_ISO_8601_UTC>"
}
```

**Response:**

```json
{
  "success": true,
  "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
  "scheduledTime": "<FUTURE_ISO_8601_UTC>"
}
```

### Get Post Status

```http
GET https://api.publora.com/api/v1/get-post/{postGroupId}
x-publora-key: sk_YOUR_API_KEY
```

**Response:**

```json
{
  "success": true,
  "postGroupId": "67a1b2c3d4e5f6a7b8c9d0e1",
  "status": "published",
  "scheduledTime": "2026-03-15T14:00:00.000Z",
  "platformSettings": {},
  "platforms": ["twitter-123456", "linkedin-ABC123"],
  "posts": [
    {
      "platform": "twitter",
      "platformId": "123456",
      "status": "published",
      "postedId": "1234567890123456789",
      "permalink": "https://twitter.com/user/status/123456"
    },
    {
      "platform": "linkedin",
      "platformId": "ABC123",
      "status": "published",
      "postedId": "urn:li:share:7000000000000000000",
      "permalink": "https://linkedin.com/posts/..."
    }
  ],
  "media": []
}
```

> **Note:** The `get-post` endpoint returns the **raw** `platformId` (e.g., `"123456"`), not the composite format (e.g., `"twitter-123456"`). The composite `platform-platformId` format is only returned by the `list-posts` and `platform-connections` endpoints.

### Get Platform Connections

```http
GET https://api.publora.com/api/v1/platform-connections
x-publora-key: sk_YOUR_API_KEY
```

**Response:**

```json
{
  "success": true,
  "connections": [
    { "platformId": "twitter-123456789", "username": "myaccount", "displayName": "My Account", "tokenStatus": "valid" },
    { "platformId": "linkedin-ABC123", "username": "My Company", "displayName": "My Company", "tokenStatus": "valid" },
    { "platformId": "threads-789012", "username": "mythreads", "displayName": "My Threads", "tokenStatus": "valid" }
  ]
}
```

Use the `platformId` field from connections as the platform identifier when creating posts.

## Examples

### JavaScript - Publora API Client

```javascript
class PubloraClient {
  constructor(apiKey) {
    this.apiKey = apiKey;
    this.baseUrl = 'https://api.publora.com/api/v1';
  }

  async request(endpoint, options = {}) {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: {
        'x-publora-key': this.apiKey,
        'Content-Type': 'application/json',
        ...options.headers
      }
    });

    if (!response.ok) {
      const error = await response.json();
      throw new Error(error.error || `HTTP ${response.status}`);
    }

    return response.json();
  }

  async createPost(content, platforms, scheduledTime) {
    return this.request('/create-post', {
      method: 'POST',
      body: JSON.stringify({ content, platforms, scheduledTime })
    });
  }

  async listPosts(params = {}) {
    const query = new URLSearchParams(params).toString();
    return this.request(`/list-posts?${query}`);
  }
}

// Usage
const client = new PubloraClient('YOUR_API_KEY');

for (let i = 0; i < 100; i++) {
  await client.createPost(
    `Post ${i + 1}`,
    ['twitter-123'],
    new Date(Date.now() + i * 3600000).toISOString()
  );
}
```

### Python - Publora API Client with Retry

```python
import requests
import time
from typing import Dict, Any, Optional

class PubloraClient:
    def __init__(self, api_key: str):
        self.api_key = api_key
        self.base_url = 'https://api.publora.com/api/v1'

    def _request(
        self,
        method: str,
        endpoint: str,
        data: Optional[Dict] = None,
        params: Optional[Dict] = None,
        max_retries: int = 3
    ) -> Dict[str, Any]:
        """Make a request with retry on failure."""

        headers = {
            'x-publora-key': self.api_key,
            'Content-Type': 'application/json'
        }

        for attempt in range(max_retries):
            response = requests.request(
                method,
                f'{self.base_url}{endpoint}',
                headers=headers,
                json=data,
                params=params
            )

            if response.status_code >= 500:
                backoff = 2 ** attempt
                print(f"Server error. Retrying in {backoff}s (attempt {attempt + 1})")
                time.sleep(backoff)
                continue

            response.raise_for_status()
            return response.json()

        raise Exception("Max retries exceeded")

    def create_post(
        self,
        content: str,
        platforms: list,
        scheduled_time: Optional[str] = None
    ) -> Dict[str, Any]:
        data = {'content': content, 'platforms': platforms}
        if scheduled_time:
            data['scheduledTime'] = scheduled_time
        return self._request('POST', '/create-post', data=data)

    def list_posts(self, **params) -> Dict[str, Any]:
        return self._request('GET', '/list-posts', params=params)


# Usage
client = PubloraClient('YOUR_API_KEY')

# Bulk create
from datetime import datetime, timedelta, timezone

base_time = datetime.now(timezone.utc) + timedelta(hours=1)

for i in range(100):
    scheduled = (base_time + timedelta(hours=i)).isoformat()
    result = client.create_post(
        f"Scheduled post {i + 1}",
        ['twitter-123', 'linkedin-ABC'],
        scheduled
    )
    print(f"Created post {i + 1}: {result['postGroupId']}")
```

## Peak Engagement Times

Optimal posting times vary by platform and audience. Use these guidelines as a starting point, then analyze your own analytics.

### Platform-Specific Best Times (UTC)

| Platform | Best Days | Best Hours (UTC) | Peak Engagement |
|----------|-----------|------------------|-----------------|
| **X/Twitter** | Tue-Thu | 13:00-16:00 | Wednesday 12:00 |
| **LinkedIn** | Tue-Thu | 10:00-12:00 | Tuesday 10:00 |
| **Instagram** | Mon, Wed | 11:00-14:00, 19:00-21:00 | Wednesday 11:00 |
| **Threads** | Tue-Thu | 12:00-15:00 | Wednesday 13:00 |
| **TikTok** | Tue-Thu | 15:00-21:00 | Thursday 19:00 |
| **YouTube** | Thu-Sat | 14:00-18:00 | Friday 15:00 |
| **Facebook** | Wed-Fri | 13:00-16:00 | Wednesday 13:00 |
| **Bluesky** | Mon-Fri | 14:00-17:00 | Tuesday 15:00 |

### JavaScript - Optimal Time Scheduler

```javascript
const OPTIMAL_TIMES = {
  twitter: {
    bestDays: [2, 3, 4], // Tue, Wed, Thu (0 = Sunday)
    bestHours: [13, 14, 15, 16],
    peakDay: 3,
    peakHour: 12
  },
  linkedin: {
    bestDays: [2, 3, 4],
    bestHours: [10, 11, 12],
    peakDay: 2,
    peakHour: 10
  },
  instagram: {
    bestDays: [1, 3], // Mon, Wed
    bestHours: [11, 12, 13, 14, 19, 20, 21],
    peakDay: 3,
    peakHour: 11
  },
  threads: {
    bestDays: [2, 3, 4],
    bestHours: [12, 13, 14, 15],
    peakDay: 3,
    peakHour: 13
  },
  tiktok: {
    bestDays: [2, 3, 4],
    bestHours: [15, 16, 17, 18, 19, 20, 21],
    peakDay: 4,
    peakHour: 19
  },
  facebook: {
    bestDays: [3, 4, 5], // Wed, Thu, Fri
    bestHours: [13, 14, 15, 16],
    peakDay: 3,
    peakHour: 13
  }
};

function getNextOptimalTime(platform, fromDate = new Date()) {
  const config = OPTIMAL_TIMES[platform];
  if (!config) throw new Error(`Unknown platform: ${platform}`);

  const result = new Date(fromDate);
  result.setUTCMinutes(0, 0, 0);

  // Find next best day
  let daysToAdd = 0;
  while (!config.bestDays.includes(result.getUTCDay()) || daysToAdd === 0) {
    result.setUTCDate(result.getUTCDate() + 1);
    daysToAdd++;
    if (daysToAdd > 7) break;
  }

  // Set to best hour
  result.setUTCHours(config.bestHours[0]);

  return result;
}

function getPeakTime(platform, weekStart = new Date()) {
  const config = OPTIMAL_TIMES[platform];
  if (!config) throw new Error(`Unknown platform: ${platform}`);

  const result = new Date(weekStart);
  result.setUTCHours(0, 0, 0, 0);

  // Move to peak day
  const currentDay = result.getUTCDay();
  const daysUntilPeak = (config.peakDay - currentDay + 7) % 7;
  result.setUTCDate(result.getUTCDate() + daysUntilPeak);

  // Set peak hour
  result.setUTCHours(config.peakHour);

  return result;
}

function distributePostsOptimally(platforms, count, startDate = new Date()) {
  const schedule = [];
  const date = new Date(startDate);

  for (let i = 0; i < count; i++) {
    const platform = platforms[i % platforms.length];
    const optimalTime = getNextOptimalTime(platform, date);

    schedule.push({
      index: i,
      platform,
      scheduledTime: optimalTime.toISOString()
    });

    // Move forward for next post
    date.setTime(optimalTime.getTime() + 3600000); // +1 hour minimum gap
  }

  return schedule;
}

// Usage
const schedule = distributePostsOptimally(
  ['twitter', 'linkedin', 'instagram'],
  9, // 3 posts per platform
  new Date()
);

console.log('Optimal posting schedule:');
schedule.forEach(item => {
  console.log(`${item.platform}: ${item.scheduledTime}`);
});
```

### Python - Queue-Based Scheduler with Rate Limits

```python
from datetime import datetime, timedelta, timezone
from typing import List, Dict, Optional
from dataclasses import dataclass
from collections import defaultdict
import heapq

@dataclass
class PlatformConfig:
    posts_per_hour: int
    posts_per_day: int
    best_hours: List[int]  # UTC hours
    best_days: List[int]   # 0=Monday, 6=Sunday

PLATFORM_LIMITS = {
    'twitter': PlatformConfig(
        posts_per_hour=10,
        posts_per_day=50,
        best_hours=[13, 14, 15, 16],
        best_days=[1, 2, 3]  # Tue, Wed, Thu
    ),
    'linkedin': PlatformConfig(
        posts_per_hour=5,
        posts_per_day=20,
        best_hours=[10, 11, 12],
        best_days=[1, 2, 3]
    ),
    'instagram': PlatformConfig(
        posts_per_hour=3,
        posts_per_day=10,
        best_hours=[11, 12, 13, 14, 19, 20, 21],
        best_days=[0, 2]  # Mon, Wed
    ),
    'tiktok': PlatformConfig(
        posts_per_hour=2,
        posts_per_day=10,
        best_hours=[15, 16, 17, 18, 19, 20, 21],
        best_days=[1, 2, 3]
    ),
}


class OptimalScheduler:
    """
    Schedule posts optimally across platforms considering:
    - Platform-specific rate limits
    - Peak engagement times
    - Even distribution across time slots
    """

    def __init__(self):
        self.platform_queues: Dict[str, List] = defaultdict(list)
        self.posts_per_hour: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))
        self.posts_per_day: Dict[str, Dict[str, int]] = defaultdict(lambda: defaultdict(int))

    def get_next_optimal_slot(
        self,
        platform: str,
        after: datetime
    ) -> datetime:
        """Find the next available optimal time slot for a platform."""
        config = PLATFORM_LIMITS.get(platform)
        if not config:
            # Unknown platform - just space by 1 hour
            return after + timedelta(hours=1)

        candidate = after.replace(minute=0, second=0, microsecond=0)

        for _ in range(168):  # Check up to 1 week ahead
            candidate += timedelta(hours=1)
            day_key = candidate.strftime('%Y-%m-%d')
            hour_key = candidate.strftime('%Y-%m-%d-%H')

            # Check rate limits
            if self.posts_per_day[platform][day_key] >= config.posts_per_day:
                # Skip to next day
                candidate = candidate.replace(hour=0) + timedelta(days=1)
                continue

            if self.posts_per_hour[platform][hour_key] >= config.posts_per_hour:
                continue

            # Check if this is an optimal time
            if (candidate.weekday() in config.best_days and
                candidate.hour in config.best_hours):
                return candidate

        # If no optimal slot found, return next available
        return after + timedelta(hours=1)

    def schedule_post(
        self,
        platform: str,
        after: datetime = None
    ) -> datetime:
        """Schedule a single post and return its time."""
        if after is None:
            after = datetime.now(timezone.utc)

        scheduled_time = self.get_next_optimal_slot(platform, after)

        # Update counters
        day_key = scheduled_time.strftime('%Y-%m-%d')
        hour_key = scheduled_time.strftime('%Y-%m-%d-%H')
        self.posts_per_day[platform][day_key] += 1
        self.posts_per_hour[platform][hour_key] += 1

        return scheduled_time

    def schedule_batch(
        self,
        posts: List[Dict],
        start_after: datetime = None
    ) -> List[Dict]:
        """
        Schedule multiple posts across platforms optimally.

        Args:
            posts: List of {'content': str, 'platforms': List[str]}
            start_after: Earliest scheduling time

        Returns:
            List of posts with scheduledTime added
        """
        if start_after is None:
            start_after = datetime.now(timezone.utc)

        scheduled_posts = []
        platform_last_time: Dict[str, datetime] = {}

        for post in posts:
            platforms = post['platforms']
            # Find the latest "next optimal time" across all platforms
            scheduled_time = start_after

            for platform in platforms:
                platform_name = platform.split('-')[0]
                after = platform_last_time.get(platform_name, start_after)
                optimal_time = self.schedule_post(platform_name, after)

                if optimal_time > scheduled_time:
                    scheduled_time = optimal_time

                platform_last_time[platform_name] = scheduled_time

            scheduled_posts.append({
                **post,
                'scheduledTime': scheduled_time.isoformat()
            })

        return scheduled_posts


# Usage example
scheduler = OptimalScheduler()

# Posts to schedule
posts = [
    {'content': 'Morning motivation!', 'platforms': ['twitter-123', 'linkedin-ABC']},
    {'content': 'Product update announcement', 'platforms': ['twitter-123', 'linkedin-ABC', 'instagram-456']},
    {'content': 'Behind the scenes', 'platforms': ['instagram-456', 'tiktok-789']},
    {'content': 'Weekly tips thread', 'platforms': ['twitter-123']},
    {'content': 'Customer success story', 'platforms': ['linkedin-ABC']},
]

scheduled = scheduler.schedule_batch(posts)

print("Optimally scheduled posts:")
for post in scheduled:
    print(f"  {post['scheduledTime']}: {post['content'][:30]}...")
    print(f"    Platforms: {post['platforms']}")
```

### Node.js - Complete Rate-Limited Queue System

```javascript
const axios = require('axios');

class PubloraQueueScheduler {
  constructor(apiKey, platformLimits = {}) {
    this.apiKey = apiKey;
    this.api = axios.create({
      baseURL: 'https://api.publora.com/api/v1',
      headers: { 'x-publora-key': apiKey }
    });

    // User-supplied throttles. Publora does not define per-platform posting quotas.
    this.platformLimits = platformLimits;

    this.peakHours = {
      twitter: [13, 14, 15, 16],
      linkedin: [10, 11, 12],
      instagram: [11, 12, 13, 14, 19, 20, 21],
      tiktok: [15, 16, 17, 18, 19, 20, 21],
      facebook: [13, 14, 15, 16],
    };

    this.postCounts = { hourly: {}, daily: {} };
  }

  getPlatformName(platformId) {
    return platformId.split('-')[0];
  }

  getHourKey(date) {
    return date.toISOString().slice(0, 13);
  }

  getDayKey(date) {
    return date.toISOString().slice(0, 10);
  }

  canPostAt(platform, date) {
    const limits = this.platformLimits[platform];
    if (!limits) return true;

    const hourKey = `${platform}-${this.getHourKey(date)}`;
    const dayKey = `${platform}-${this.getDayKey(date)}`;

    const hourlyCount = this.postCounts.hourly[hourKey] || 0;
    const dailyCount = this.postCounts.daily[dayKey] || 0;

    return hourlyCount < limits.perHour && dailyCount < limits.perDay;
  }

  recordPost(platform, date) {
    const hourKey = `${platform}-${this.getHourKey(date)}`;
    const dayKey = `${platform}-${this.getDayKey(date)}`;

    this.postCounts.hourly[hourKey] = (this.postCounts.hourly[hourKey] || 0) + 1;
    this.postCounts.daily[dayKey] = (this.postCounts.daily[dayKey] || 0) + 1;
  }

  isPeakTime(platform, date) {
    const hours = this.peakHours[platform];
    return hours ? hours.includes(date.getUTCHours()) : true;
  }

  findNextOptimalSlot(platforms, after) {
    let candidate = new Date(after);
    candidate.setUTCMinutes(0, 0, 0);

    for (let i = 0; i < 168; i++) { // Check up to 1 week
      candidate = new Date(candidate.getTime() + 3600000); // +1 hour

      const allCanPost = platforms.every(p =>
        this.canPostAt(this.getPlatformName(p), candidate)
      );

      if (!allCanPost) continue;

      const anyPeak = platforms.some(p =>
        this.isPeakTime(this.getPlatformName(p), candidate)
      );

      if (anyPeak) {
        return candidate;
      }
    }

    // Fallback: return next hour
    return new Date(after.getTime() + 3600000);
  }

  async scheduleOptimally(posts) {
    const results = [];
    let lastTime = new Date();

    for (const post of posts) {
      const scheduledTime = this.findNextOptimalSlot(post.platforms, lastTime);

      // Record for rate limiting
      post.platforms.forEach(p => {
        this.recordPost(this.getPlatformName(p), scheduledTime);
      });

      try {
        const { data } = await this.api.post('/create-post', {
          content: post.content,
          platforms: post.platforms,
          scheduledTime: scheduledTime.toISOString()
        });

        results.push({
          success: true,
          postGroupId: data.postGroupId,
          scheduledTime: scheduledTime.toISOString(),
          content: post.content.slice(0, 50)
        });

        console.log(`Scheduled: ${post.content.slice(0, 30)}... at ${scheduledTime.toISOString()}`);
      } catch (error) {
        results.push({
          success: false,
          error: error.response?.data?.message || error.message,
          content: post.content.slice(0, 50)
        });
      }

      lastTime = scheduledTime;

      // Small delay between API calls
      await new Promise(r => setTimeout(r, 100));
    }

    return results;
  }
}

// Usage
const scheduler = new PubloraQueueScheduler('YOUR_API_KEY');

const posts = [
  { content: 'Morning motivation!', platforms: ['twitter-123', 'linkedin-ABC'] },
  { content: 'Product update', platforms: ['twitter-123', 'linkedin-ABC', 'instagram-456'] },
  { content: 'Behind the scenes', platforms: ['instagram-456', 'tiktok-789'] },
];

const results = await scheduler.scheduleOptimally(posts);
console.log(`Scheduled ${results.filter(r => r.success).length} of ${results.length} posts`);
```

## Complete Queue-Based Scheduling System

This section provides a production-ready implementation that combines all the concepts: rate limiting, optimal scheduling, error handling, retry logic, and failed post recovery.

### Architecture Overview

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│  Content Queue  │───►│  Rate Limiter    │───►│  Publora API    │
│  (your posts)   │    │  + Scheduler     │    │                 │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │                        │
                              ▼                        ▼
                       ┌──────────────────┐    ┌─────────────────┐
                       │  Retry Queue     │    │  Status Checker │
                       │  (failed posts)  │    │  (polling)      │
                       └──────────────────┘    └─────────────────┘
```

### JavaScript - Production Queue Scheduler

```javascript
const axios = require('axios');

/**
 * Production-ready queue-based scheduler for Publora API
 * Features:
 * - Platform-specific rate limiting
 * - Peak engagement time optimization
 * - Exponential backoff retry logic
 * - Failed post recovery
 * - Comprehensive error handling
 */
class PubloraQueueScheduler {
  constructor(apiKey, options = {}) {
    this.apiKey = apiKey;
    this.baseUrl = options.baseUrl || 'https://api.publora.com/api/v1';
    this.maxRetries = options.maxRetries || 3;
    this.retryDelay = options.retryDelay || 1000;

    // API rate limit tracking
    this.rateLimitRemaining = Infinity;
    this.rateLimitReset = 0;

    // Optional user-supplied throttles; Publora defines no per-platform quotas.
    this.platformLimits = options.platformLimits || {};

    // Peak engagement hours (UTC) - all 10 platforms
    this.peakHours = {
      twitter:   [13, 14, 15, 16],
      linkedin:  [10, 11, 12],
      instagram: [11, 12, 13, 14, 19, 20, 21],
      threads:   [12, 13, 14, 15],
      tiktok:    [15, 16, 17, 18, 19, 20, 21],
      facebook:  [13, 14, 15, 16],
      youtube:   [14, 15, 16, 17, 18],
      bluesky:   [14, 15, 16, 17],
      mastodon:  [14, 15, 16, 17],
      telegram:  [9, 10, 11, 12, 18, 19, 20],
    };

    // Track scheduled posts per platform
    this.postCounts = { hourly: {}, daily: {} };

    // Failed posts queue for retry
    this.failedQueue = [];

    // Results tracking
    this.results = [];
  }

  // ... (rest of the implementation)
}
```

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
