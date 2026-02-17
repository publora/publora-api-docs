# Bulk Scheduling Guide

Schedule multiple posts at once using batch operations, CSV imports, and content calendars.

## Overview

Bulk scheduling allows you to:
- Schedule a week or month of content in one script
- Import posts from CSV or spreadsheets
- Create content calendars with automated publishing
- Batch update or delete multiple posts

## Schedule Multiple Posts

### JavaScript: Schedule a Week of Content

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

const headers = {
  'Content-Type': 'application/json',
  'x-publora-key': PUBLORA_API_KEY
};

async function scheduleWeekOfContent(posts, platforms, startDate = new Date()) {
  const results = [];

  for (let i = 0; i < posts.length; i++) {
    // Schedule each post on a different day at 10 AM UTC
    const scheduledDate = new Date(startDate);
    scheduledDate.setDate(startDate.getDate() + i);
    scheduledDate.setUTCHours(10, 0, 0, 0);

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        content: posts[i].content,
        platforms,
        scheduledTime: scheduledDate.toISOString()
      })
    });

    const result = await response.json();
    results.push({
      day: i + 1,
      date: scheduledDate.toDateString(),
      postGroupId: result.postGroupId,
      success: response.ok
    });

    console.log(`Day ${i + 1}: ${response.ok ? '✓' : '✗'} ${scheduledDate.toDateString()}`);

    // Small delay to avoid rate limits
    await new Promise(r => setTimeout(r, 200));
  }

  return results;
}

// Example: 7 days of content
const weeklyPosts = [
  { content: 'Monday motivation: Start your week with intention!' },
  { content: 'Tech tip Tuesday: Always version your APIs.' },
  { content: 'Wednesday wisdom: Ship fast, iterate faster.' },
  { content: 'Throwback Thursday: How we grew from 0 to 10K users.' },
  { content: 'Feature Friday: Check out our new analytics dashboard!' },
  { content: 'Saturday reading: Top 5 dev articles this week.' },
  { content: 'Sunday reflection: What will you build next week?' }
];

scheduleWeekOfContent(weeklyPosts, ['twitter-123456', 'linkedin-ABC123']);
```

### Python: Schedule a Month of Content

```python
import requests
from datetime import datetime, timedelta
import time

PUBLORA_API_KEY = 'YOUR_API_KEY'
BASE_URL = 'https://api.publora.com/api/v1'

headers = {
    'Content-Type': 'application/json',
    'x-publora-key': PUBLORA_API_KEY
}

def schedule_month_of_content(posts, platforms, start_date=None):
    if start_date is None:
        start_date = datetime.utcnow()

    results = []

    for i, post in enumerate(posts):
        scheduled_date = start_date + timedelta(days=i)
        scheduled_date = scheduled_date.replace(hour=10, minute=0, second=0, microsecond=0)

        response = requests.post(
            f'{BASE_URL}/create-post',
            headers=headers,
            json={
                'content': post['content'],
                'platforms': platforms,
                'scheduledTime': scheduled_date.isoformat() + 'Z'
            }
        )

        result = {
            'day': i + 1,
            'date': scheduled_date.strftime('%Y-%m-%d'),
            'success': response.ok
        }

        if response.ok:
            result['postGroupId'] = response.json()['postGroupId']

        results.append(result)
        print(f"Day {i + 1}: {'✓' if response.ok else '✗'} {scheduled_date.strftime('%Y-%m-%d')}")

        time.sleep(0.2)  # Rate limiting

    successful = sum(1 for r in results if r['success'])
    print(f"\nScheduled {successful}/{len(posts)} posts")

    return results

# Example: Generate 30 days of content
monthly_posts = [
    {'content': f'Day {i+1} of our 30-day challenge! #30daychallenge'}
    for i in range(30)
]

schedule_month_of_content(monthly_posts, ['twitter-123456'])
```

## Import from CSV

### CSV Format

Create a CSV file with your content:

```csv
content,platforms,scheduled_time
"Monday motivation: Start strong!",twitter-123456;linkedin-ABC123,2026-03-01T09:00:00Z
"Check out our new feature!",twitter-123456,2026-03-01T14:00:00Z
"Weekly roundup thread",twitter-123456,2026-03-02T10:00:00Z
"LinkedIn deep dive post",linkedin-ABC123,2026-03-02T12:00:00Z
"Behind the scenes",instagram-789012,2026-03-03T11:00:00Z
```

### JavaScript CSV Import

```javascript
const fs = require('fs');
const csv = require('csv-parse/sync');

async function importFromCSV(filePath) {
  const fileContent = fs.readFileSync(filePath, 'utf-8');
  const records = csv.parse(fileContent, {
    columns: true,
    skip_empty_lines: true
  });

  const results = [];

  for (const row of records) {
    const platforms = row.platforms.split(';').map(p => p.trim());

    const payload = {
      content: row.content,
      platforms,
      scheduledTime: row.scheduled_time
    };

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers,
      body: JSON.stringify(payload)
    });

    const result = await response.json();

    results.push({
      content: row.content.substring(0, 30) + '...',
      success: response.ok,
      postGroupId: result.postGroupId
    });

    console.log(`${response.ok ? '✓' : '✗'} ${row.content.substring(0, 40)}...`);

    await new Promise(r => setTimeout(r, 200));
  }

  const successful = results.filter(r => r.success).length;
  console.log(`\nImported ${successful}/${results.length} posts`);

  return results;
}

importFromCSV('./posts.csv');
```

### Python CSV Import

```python
import csv
import requests
import time

def import_from_csv(file_path):
    results = []

    with open(file_path, 'r', encoding='utf-8') as f:
        reader = csv.DictReader(f)

        for row in reader:
            platforms = [p.strip() for p in row['platforms'].split(';')]

            payload = {
                'content': row['content'],
                'platforms': platforms,
                'scheduledTime': row['scheduled_time']
            }

            response = requests.post(
                f'{BASE_URL}/create-post',
                headers=headers,
                json=payload
            )

            result = {
                'content': row['content'][:30] + '...',
                'success': response.ok
            }

            if response.ok:
                result['postGroupId'] = response.json()['postGroupId']
            else:
                result['error'] = response.json().get('message', 'Unknown error')

            results.append(result)
            status = '✓' if response.ok else '✗'
            print(f"{status} {row['content'][:40]}...")

            time.sleep(0.2)

    successful = sum(1 for r in results if r['success'])
    print(f"\nImported {successful}/{len(results)} posts")

    return results

import_from_csv('posts.csv')
```

## Content Calendar Patterns

### Schedule at Optimal Times

```javascript
// Optimal posting times by platform (UTC)
const OPTIMAL_TIMES = {
  twitter: [9, 12, 17],    // 9 AM, 12 PM, 5 PM
  linkedin: [7, 10, 17],   // 7 AM, 10 AM, 5 PM
  instagram: [11, 14, 19], // 11 AM, 2 PM, 7 PM
  facebook: [9, 13, 16]    // 9 AM, 1 PM, 4 PM
};

function getOptimalTime(platform, dayOffset = 0) {
  const times = OPTIMAL_TIMES[platform] || [10];
  const randomTime = times[Math.floor(Math.random() * times.length)];

  const date = new Date();
  date.setDate(date.getDate() + dayOffset);
  date.setUTCHours(randomTime, 0, 0, 0);

  return date;
}

async function scheduleAtOptimalTimes(posts) {
  for (let i = 0; i < posts.length; i++) {
    const post = posts[i];

    for (const platformId of post.platforms) {
      const platform = platformId.split('-')[0];
      const optimalTime = getOptimalTime(platform, i);

      await fetch(`${BASE_URL}/create-post`, {
        method: 'POST',
        headers,
        body: JSON.stringify({
          content: post.content,
          platforms: [platformId],
          scheduledTime: optimalTime.toISOString()
        })
      });

      console.log(`Scheduled for ${platform} at ${optimalTime.toISOString()}`);
    }
  }
}
```

### Stagger Posts Across Platforms

```javascript
async function staggerAcrossPlatforms(content, platforms, baseTime, intervalMinutes = 30) {
  const results = [];

  for (let i = 0; i < platforms.length; i++) {
    const scheduledTime = new Date(baseTime);
    scheduledTime.setMinutes(scheduledTime.getMinutes() + (i * intervalMinutes));

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        content,
        platforms: [platforms[i]],
        scheduledTime: scheduledTime.toISOString()
      })
    });

    results.push({
      platform: platforms[i],
      scheduledTime: scheduledTime.toISOString(),
      success: response.ok
    });
  }

  return results;
}

// Post to Twitter at 10:00, LinkedIn at 10:30, Threads at 11:00
staggerAcrossPlatforms(
  'Exciting announcement coming!',
  ['twitter-123456', 'linkedin-ABC123', 'threads-789012'],
  new Date('2026-03-01T10:00:00Z'),
  30
);
```

### Weekly Recurring Schedule

```javascript
async function createWeeklyRecurring(content, platforms, dayOfWeek, timeUTC, weeks = 4) {
  const results = [];

  // Find next occurrence of dayOfWeek (0 = Sunday, 1 = Monday, etc.)
  const startDate = new Date();
  while (startDate.getDay() !== dayOfWeek) {
    startDate.setDate(startDate.getDate() + 1);
  }
  startDate.setUTCHours(timeUTC, 0, 0, 0);

  for (let i = 0; i < weeks; i++) {
    const scheduledTime = new Date(startDate);
    scheduledTime.setDate(startDate.getDate() + (i * 7));

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        content,
        platforms,
        scheduledTime: scheduledTime.toISOString()
      })
    });

    results.push({
      week: i + 1,
      scheduledTime: scheduledTime.toISOString(),
      success: response.ok
    });

    console.log(`Week ${i + 1}: ${scheduledTime.toDateString()}`);
  }

  return results;
}

// Post every Monday at 9 AM UTC for 4 weeks
createWeeklyRecurring(
  'Happy Monday! What are your goals this week?',
  ['twitter-123456', 'linkedin-ABC123'],
  1, // Monday
  9, // 9 AM UTC
  4  // 4 weeks
);
```

## Batch Operations

### Batch Update Scheduled Times

```javascript
async function batchReschedule(postGroupIds, newBaseTime, intervalHours = 2) {
  const results = [];

  for (let i = 0; i < postGroupIds.length; i++) {
    const newTime = new Date(newBaseTime);
    newTime.setHours(newTime.getHours() + (i * intervalHours));

    const response = await fetch(`${BASE_URL}/update-post/${postGroupIds[i]}`, {
      method: 'PUT',
      headers,
      body: JSON.stringify({
        scheduledTime: newTime.toISOString()
      })
    });

    results.push({
      postGroupId: postGroupIds[i],
      newTime: newTime.toISOString(),
      success: response.ok
    });
  }

  return results;
}

// Reschedule all posts starting from a new date
batchReschedule(
  ['pg_abc123', 'pg_def456', 'pg_ghi789'],
  new Date('2026-03-15T10:00:00Z'),
  2 // 2 hours apart
);
```

### Batch Delete

```javascript
async function batchDelete(postGroupIds) {
  const results = [];

  for (const postGroupId of postGroupIds) {
    const response = await fetch(`${BASE_URL}/delete-post/${postGroupId}`, {
      method: 'DELETE',
      headers: { 'x-publora-key': PUBLORA_API_KEY }
    });

    results.push({
      postGroupId,
      success: response.ok
    });

    await new Promise(r => setTimeout(r, 100));
  }

  const successful = results.filter(r => r.success).length;
  console.log(`Deleted ${successful}/${results.length} posts`);

  return results;
}
```

### Convert Drafts to Scheduled

```javascript
async function publishAllDrafts(draftIds, startTime, intervalMinutes = 60) {
  const results = [];

  for (let i = 0; i < draftIds.length; i++) {
    const scheduledTime = new Date(startTime);
    scheduledTime.setMinutes(scheduledTime.getMinutes() + (i * intervalMinutes));

    const response = await fetch(`${BASE_URL}/update-post/${draftIds[i]}`, {
      method: 'PUT',
      headers,
      body: JSON.stringify({
        status: 'scheduled',
        scheduledTime: scheduledTime.toISOString()
      })
    });

    results.push({
      postGroupId: draftIds[i],
      scheduledTime: scheduledTime.toISOString(),
      success: response.ok
    });
  }

  return results;
}
```

## Rate Limiting Best Practices

```javascript
class RateLimitedScheduler {
  constructor(apiKey, requestsPerSecond = 5) {
    this.apiKey = apiKey;
    this.minInterval = 1000 / requestsPerSecond;
    this.lastRequest = 0;
  }

  async schedule(post) {
    // Wait if needed to respect rate limit
    const now = Date.now();
    const elapsed = now - this.lastRequest;

    if (elapsed < this.minInterval) {
      await new Promise(r => setTimeout(r, this.minInterval - elapsed));
    }

    this.lastRequest = Date.now();

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': this.apiKey
      },
      body: JSON.stringify(post)
    });

    return response.json();
  }

  async scheduleBatch(posts) {
    const results = [];

    for (const post of posts) {
      const result = await this.schedule(post);
      results.push(result);
    }

    return results;
  }
}

// Usage
const scheduler = new RateLimitedScheduler(PUBLORA_API_KEY, 5);

const posts = [
  { content: 'Post 1', platforms: ['twitter-123456'], scheduledTime: '2026-03-01T10:00:00Z' },
  { content: 'Post 2', platforms: ['twitter-123456'], scheduledTime: '2026-03-01T12:00:00Z' },
  { content: 'Post 3', platforms: ['twitter-123456'], scheduledTime: '2026-03-01T14:00:00Z' }
];

scheduler.scheduleBatch(posts);
```

## Error Handling

```javascript
async function scheduleWithRetry(post, maxRetries = 3) {
  for (let attempt = 1; attempt <= maxRetries; attempt++) {
    try {
      const response = await fetch(`${BASE_URL}/create-post`, {
        method: 'POST',
        headers,
        body: JSON.stringify(post)
      });

      if (response.ok) {
        return { success: true, data: await response.json() };
      }

      if (response.status === 429) {
        // Rate limited - wait and retry
        const retryAfter = response.headers.get('Retry-After') || 5;
        console.log(`Rate limited. Waiting ${retryAfter}s before retry ${attempt}/${maxRetries}`);
        await new Promise(r => setTimeout(r, retryAfter * 1000));
        continue;
      }

      const error = await response.json();
      return { success: false, error: error.error || error.message };

    } catch (error) {
      if (attempt === maxRetries) {
        return { success: false, error: error.error || error.message };
      }
      await new Promise(r => setTimeout(r, 1000 * attempt));
    }
  }

  return { success: false, error: 'Max retries exceeded' };
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
