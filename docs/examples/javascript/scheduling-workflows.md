# JavaScript Scheduling Workflows

Advanced scheduling patterns for automating your social media content.

## Schedule a Week of Content

```javascript
const PUBLORA_API_KEY = 'YOUR_API_KEY';
const BASE_URL = 'https://api.publora.com/api/v1';

const headers = {
  'Content-Type': 'application/json',
  'x-publora-key': PUBLORA_API_KEY
};

async function scheduleWeekOfContent(posts, platforms) {
  const results = [];
  const startDate = new Date();

  for (let i = 0; i < posts.length; i++) {
    const post = posts[i];

    // Schedule each post on a different day at 10 AM UTC
    const scheduledDate = new Date(startDate);
    scheduledDate.setDate(startDate.getDate() + i);
    scheduledDate.setUTCHours(10, 0, 0, 0);

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        content: post.content,
        platforms,
        scheduledTime: scheduledDate.toISOString(),
        mediaUrls: post.mediaUrls || []
      })
    });

    const result = await response.json();
    results.push({
      day: i + 1,
      postGroupId: result.postGroupId,
      scheduledTime: scheduledDate.toISOString()
    });

    console.log(`Day ${i + 1}: Scheduled for ${scheduledDate.toDateString()}`);
  }

  return results;
}

// Usage: Schedule 7 days of content
const weeklyPosts = [
  { content: 'Monday motivation: Start your week strong!' },
  { content: 'Tech tip Tuesday: Always version your APIs.' },
  { content: 'Wednesday wisdom: Ship fast, iterate faster.' },
  { content: 'Throwback Thursday: Our journey from 0 to 10K users.' },
  { content: 'Feature Friday: Check out our new dashboard!' },
  { content: 'Saturday reading list: Top 5 dev blogs this week.' },
  { content: 'Sunday reflection: What will you build this week?' }
];

await scheduleWeekOfContent(weeklyPosts, ['twitter-123456', 'linkedin-ABC123']);
```

## Schedule at Optimal Times

```javascript
// Optimal posting times by platform (in UTC)
const OPTIMAL_TIMES = {
  twitter: [9, 12, 17], // 9 AM, 12 PM, 5 PM
  linkedin: [7, 10, 17], // 7 AM, 10 AM, 5 PM
  instagram: [11, 14, 19], // 11 AM, 2 PM, 7 PM
  facebook: [9, 13, 16] // 9 AM, 1 PM, 4 PM
};

function getNextOptimalTime(platform, afterDate = new Date()) {
  const times = OPTIMAL_TIMES[platform] || [10];
  const result = new Date(afterDate);

  // Find next optimal hour today or tomorrow
  for (const hour of times) {
    result.setUTCHours(hour, 0, 0, 0);
    if (result > afterDate) {
      return result;
    }
  }

  // If all times today have passed, use first time tomorrow
  result.setDate(result.getDate() + 1);
  result.setUTCHours(times[0], 0, 0, 0);
  return result;
}

async function scheduleAtOptimalTime(content, platformId) {
  const platform = platformId.split('-')[0]; // Extract platform name
  const optimalTime = getNextOptimalTime(platform);

  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      content,
      platforms: [platformId],
      scheduledTime: optimalTime.toISOString()
    })
  });

  console.log(`Scheduled for ${platform} at ${optimalTime.toISOString()}`);
  return response.json();
}

// Usage
await scheduleAtOptimalTime('Check out our latest update!', 'twitter-123456');
await scheduleAtOptimalTime('Check out our latest update!', 'linkedin-ABC123');
```

## Batch Schedule from Array

```javascript
async function batchSchedule(posts, delayBetweenMs = 1000) {
  const results = [];

  for (const post of posts) {
    try {
      const response = await fetch(`${BASE_URL}/create-post`, {
        method: 'POST',
        headers,
        body: JSON.stringify(post)
      });

      const result = await response.json();

      if (response.ok) {
        results.push({ success: true, postGroupId: result.postGroupId, post });
      } else {
        results.push({ success: false, error: result, post });
      }
    } catch (error) {
      results.push({ success: false, error: error.message, post });
    }

    // Delay between requests to avoid rate limits
    await new Promise(resolve => setTimeout(resolve, delayBetweenMs));
  }

  // Summary
  const successful = results.filter(r => r.success).length;
  console.log(`Scheduled ${successful}/${posts.length} posts successfully`);

  return results;
}

// Usage
const postsToSchedule = [
  {
    content: 'Post 1: Morning update',
    platforms: ['twitter-123456'],
    scheduledTime: '2026-03-01T09:00:00.000Z'
  },
  {
    content: 'Post 2: Afternoon engagement',
    platforms: ['twitter-123456', 'linkedin-ABC123'],
    scheduledTime: '2026-03-01T14:00:00.000Z'
  },
  {
    content: 'Post 3: Evening wrap-up',
    platforms: ['twitter-123456'],
    scheduledTime: '2026-03-01T18:00:00.000Z'
  }
];

await batchSchedule(postsToSchedule);
```

## Schedule Recurring Content

```javascript
async function scheduleRecurring(content, platforms, options) {
  const {
    startDate = new Date(),
    frequency = 'weekly', // 'daily', 'weekly', 'monthly'
    count = 4, // Number of occurrences
    timeOfDay = 10 // Hour in UTC
  } = options;

  const results = [];

  for (let i = 0; i < count; i++) {
    const scheduledDate = new Date(startDate);

    switch (frequency) {
      case 'daily':
        scheduledDate.setDate(startDate.getDate() + i);
        break;
      case 'weekly':
        scheduledDate.setDate(startDate.getDate() + (i * 7));
        break;
      case 'monthly':
        scheduledDate.setMonth(startDate.getMonth() + i);
        break;
    }

    scheduledDate.setUTCHours(timeOfDay, 0, 0, 0);

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers,
      body: JSON.stringify({
        content,
        platforms,
        scheduledTime: scheduledDate.toISOString()
      })
    });

    results.push(await response.json());
  }

  return results;
}

// Schedule weekly reminder for 4 weeks
await scheduleRecurring(
  'Reminder: Our weekly webinar is tomorrow at 2 PM EST!',
  ['twitter-123456', 'linkedin-ABC123'],
  { frequency: 'weekly', count: 4, timeOfDay: 14 }
);
```

## Schedule with Time Zone Conversion

```javascript
function toUTC(localTime, timezone) {
  // Convert local time to UTC
  const date = new Date(localTime);
  const utcDate = new Date(date.toLocaleString('en-US', { timeZone: 'UTC' }));
  const tzDate = new Date(date.toLocaleString('en-US', { timeZone: timezone }));
  const offset = tzDate - utcDate;
  return new Date(date.getTime() - offset);
}

async function scheduleInTimezone(content, platforms, localTime, timezone) {
  const utcTime = toUTC(localTime, timezone);

  const response = await fetch(`${BASE_URL}/create-post`, {
    method: 'POST',
    headers,
    body: JSON.stringify({
      content,
      platforms,
      scheduledTime: utcTime.toISOString()
    })
  });

  console.log(`Local: ${localTime} (${timezone})`);
  console.log(`UTC: ${utcTime.toISOString()}`);

  return response.json();
}

// Schedule for 9 AM Eastern Time
await scheduleInTimezone(
  'Good morning East Coast!',
  ['twitter-123456'],
  '2026-03-01T09:00:00',
  'America/New_York'
);

// Schedule for 9 AM Pacific Time
await scheduleInTimezone(
  'Good morning West Coast!',
  ['twitter-123456'],
  '2026-03-01T09:00:00',
  'America/Los_Angeles'
);
```

## Queue Management

```javascript
class PostQueue {
  constructor(apiKey, rateLimit = 10) {
    this.apiKey = apiKey;
    this.rateLimit = rateLimit;
    this.queue = [];
    this.processing = false;
  }

  add(post) {
    this.queue.push(post);
    if (!this.processing) {
      this.process();
    }
  }

  async process() {
    this.processing = true;

    while (this.queue.length > 0) {
      const batch = this.queue.splice(0, this.rateLimit);

      const promises = batch.map(post =>
        fetch(`${BASE_URL}/create-post`, {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            'x-publora-key': this.apiKey
          },
          body: JSON.stringify(post)
        }).then(r => r.json())
      );

      const results = await Promise.all(promises);
      console.log(`Processed ${results.length} posts`);

      // Wait before next batch
      if (this.queue.length > 0) {
        await new Promise(r => setTimeout(r, 1000));
      }
    }

    this.processing = false;
  }
}

// Usage
const queue = new PostQueue(PUBLORA_API_KEY);

// Add posts to queue - they'll be processed automatically
queue.add({ content: 'Post 1', platforms: ['twitter-123456'] });
queue.add({ content: 'Post 2', platforms: ['twitter-123456'] });
queue.add({ content: 'Post 3', platforms: ['twitter-123456'] });
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
