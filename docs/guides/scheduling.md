# Scheduling Posts

This guide covers how to schedule social media posts through the Publora API, including creating drafts, scheduling for specific times, and managing post lifecycles.

## How It Works

Every post in Publora follows a lifecycle defined by its status:

```
draft --> scheduled --> processing --> published
                                  \-> failed
                                  \-> partially_published
```

- **draft** -- A post saved without a `scheduledTime`. It will not be published until you update it with a valid time.
- **scheduled** -- A post with a future `scheduledTime`. The Publora scheduler checks for due posts every minute.
- **processing** -- The post's scheduled time has arrived and Publora is actively publishing it to the target platforms.
- **published** -- All platform posts succeeded.
- **failed** -- All platform posts failed.
- **partially_published** -- Some platform posts succeeded while others failed.

### Key Rules

- `scheduledTime` must be in **ISO 8601 UTC format** (e.g., `2026-03-15T14:30:00.000Z`).
- The time must be in the future. Passing a time in the past returns a `400` error.
- **Free trial users** (14-day free trial) can have a maximum of **5 pending (scheduled) posts** at any time.
- The scheduler runs every minute, so posts are published within roughly 60 seconds of their scheduled time.

## Examples

### Schedule a Post for a Specific Time

**JavaScript (fetch)**

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Excited to announce our new product launch!',
    platforms: ['twitter-123', 'linkedin-ABC'],
    scheduledTime: '2026-03-15T14:30:00.000Z'
  })
});

const data = await response.json();
console.log('Post group created:', data.postGroupId);
```

**Python (requests)**

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Excited to announce our new product launch!',
        'platforms': ['twitter-123', 'linkedin-ABC'],
        'scheduledTime': '2026-03-15T14:30:00.000Z'
    }
)

data = response.json()
print(f"Post group created: {data['postGroupId']}")
```

**cURL**

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Excited to announce our new product launch!",
    "platforms": ["twitter-123", "linkedin-ABC"],
    "scheduledTime": "2026-03-15T14:30:00.000Z"
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const { data } = await axios.post(
  'https://api.publora.com/api/v1/create-post',
  {
    content: 'Excited to announce our new product launch!',
    platforms: ['twitter-123', 'linkedin-ABC'],
    scheduledTime: '2026-03-15T14:30:00.000Z'
  },
  {
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    }
  }
);

console.log('Post group created:', data.postGroupId);
```

---

### Create a Draft, Then Schedule It Later

Sometimes you want to prepare content first and schedule it at a later time. Omit `scheduledTime` to create a draft, then update it with a `PUT` request.

**JavaScript (fetch)**

```javascript
// Step 1: Create a draft (no scheduledTime)
const draftResponse = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Draft content that needs review before publishing.',
    platforms: ['twitter-123', 'linkedin-ABC']
    // No scheduledTime -- this creates a draft
  })
});

const draft = await draftResponse.json();
console.log('Draft created:', draft.postGroupId);

// Step 2: After review, schedule the draft
const scheduleResponse = await fetch(
  `https://api.publora.com/api/v1/update-post/${draft.postGroupId}`,
  {
    method: 'PUT',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      scheduledTime: '2026-03-20T09:00:00.000Z'
    })
  }
);

const scheduled = await scheduleResponse.json();
console.log('Now scheduled:', scheduled.postGroup.status); // "scheduled"
```

**Python (requests)**

```python
import requests

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

# Step 1: Create a draft
draft_response = requests.post(
    f'{API_URL}/create-post',
    headers=HEADERS,
    json={
        'content': 'Draft content that needs review before publishing.',
        'platforms': ['twitter-123', 'linkedin-ABC']
    }
)

draft = draft_response.json()
print(f"Draft created: {draft['postGroupId']}")

# Step 2: Schedule the draft
schedule_response = requests.put(
    f"{API_URL}/update-post/{draft['postGroupId']}",
    headers=HEADERS,
    json={
        'scheduledTime': '2026-03-20T09:00:00.000Z'
    }
)

scheduled = schedule_response.json()
print(f"Now scheduled: {scheduled['postGroup']['status']}")  # "scheduled"
```

**cURL**

```bash
# Step 1: Create a draft
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Draft content that needs review before publishing.",
    "platforms": ["twitter-123", "linkedin-ABC"]
  }'

# Step 2: Schedule it (replace POST_GROUP_ID with the postGroupId from step 1)
curl -X PUT https://api.publora.com/api/v1/update-post/POST_GROUP_ID \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "scheduledTime": "2026-03-20T09:00:00.000Z"
  }'
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

// Step 1: Create a draft
const { data: draft } = await api.post('/create-post', {
  content: 'Draft content that needs review before publishing.',
  platforms: ['twitter-123', 'linkedin-ABC']
});

console.log('Draft created:', draft.postGroupId);

// Step 2: Schedule the draft
const { data: scheduled } = await api.put(`/update-post/${draft.postGroupId}`, {
  scheduledTime: '2026-03-20T09:00:00.000Z'
});

console.log('Now scheduled:', scheduled.postGroup.status); // "scheduled"
```

---

### Batch Schedule a Week of Content

You can loop through a set of posts and schedule them across an entire week. This is useful for content calendars and automated pipelines.

**JavaScript (fetch)**

```javascript
const posts = [
  { content: 'Monday motivation: Start the week strong!', day: 1 },
  { content: 'Tuesday tip: Automate your social media with Publora.', day: 2 },
  { content: 'Wednesday wisdom: Consistency beats perfection.', day: 3 },
  { content: 'Thursday thought: Your audience is waiting.', day: 4 },
  { content: 'Friday feature: Check out our new analytics dashboard!', day: 5 },
  { content: 'Saturday story: How we built Publora.', day: 6 },
  { content: 'Sunday summary: Week in review.', day: 7 }
];

const baseDate = new Date('2026-03-16T10:00:00.000Z'); // Monday at 10 AM UTC

const results = [];

for (const post of posts) {
  const scheduledTime = new Date(baseDate);
  scheduledTime.setUTCDate(baseDate.getUTCDate() + (post.day - 1));

  const response = await fetch('https://api.publora.com/api/v1/create-post', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': 'YOUR_API_KEY'
    },
    body: JSON.stringify({
      content: post.content,
      platforms: ['twitter-123', 'linkedin-ABC'],
      scheduledTime: scheduledTime.toISOString()
    })
  });

  const data = await response.json();
  results.push({ postGroupId: data.postGroupId, scheduledTime: scheduledTime.toISOString() });
  console.log(`Scheduled "${post.content.slice(0, 30)}..." for ${scheduledTime.toISOString()}`);
}

console.log(`Successfully scheduled ${results.length} posts for the week.`);
```

**Python (requests)**

```python
import requests
from datetime import datetime, timedelta, timezone

API_URL = 'https://api.publora.com/api/v1'
HEADERS = {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
}

posts = [
    {'content': 'Monday motivation: Start the week strong!', 'day': 0},
    {'content': 'Tuesday tip: Automate your social media with Publora.', 'day': 1},
    {'content': 'Wednesday wisdom: Consistency beats perfection.', 'day': 2},
    {'content': 'Thursday thought: Your audience is waiting.', 'day': 3},
    {'content': 'Friday feature: Check out our new analytics dashboard!', 'day': 4},
    {'content': 'Saturday story: How we built Publora.', 'day': 5},
    {'content': 'Sunday summary: Week in review.', 'day': 6},
]

base_date = datetime(2026, 3, 16, 10, 0, 0, tzinfo=timezone.utc)  # Monday 10 AM UTC

results = []

for post in posts:
    scheduled_time = base_date + timedelta(days=post['day'])

    response = requests.post(
        f'{API_URL}/create-post',
        headers=HEADERS,
        json={
            'content': post['content'],
            'platforms': ['twitter-123', 'linkedin-ABC'],
            'scheduledTime': scheduled_time.isoformat()
        }
    )

    data = response.json()
    results.append({'postGroupId': data['postGroupId'], 'scheduledTime': scheduled_time.isoformat()})
    print(f"Scheduled: {post['content'][:30]}... for {scheduled_time.isoformat()}")

print(f"Successfully scheduled {len(results)} posts for the week.")
```

**cURL**

```bash
#!/bin/bash

API_KEY="YOUR_API_KEY"
BASE_URL="https://api.publora.com/api/v1/create-post"

TEXTS=(
  "Monday motivation: Start the week strong!"
  "Tuesday tip: Automate your social media with Publora."
  "Wednesday wisdom: Consistency beats perfection."
  "Thursday thought: Your audience is waiting."
  "Friday feature: Check out our new analytics dashboard!"
  "Saturday story: How we built Publora."
  "Sunday summary: Week in review."
)

# Base date: Monday 2026-03-16 at 10:00 UTC
for i in "${!TEXTS[@]}"; do
  SCHEDULED_TIME=$(date -u -d "2026-03-16 10:00:00 UTC + $i days" +"%Y-%m-%dT%H:%M:%S.000Z")

  curl -s -X POST "$BASE_URL" \
    -H "Content-Type: application/json" \
    -H "x-publora-key: $API_KEY" \
    -d "{
      \"content\": \"${TEXTS[$i]}\",
      \"platforms\": [\"twitter-123\", \"linkedin-ABC\"],
      \"scheduledTime\": \"$SCHEDULED_TIME\"
    }"

  echo "Scheduled day $((i+1)): $SCHEDULED_TIME"
done
```

**Node.js (axios)**

```javascript
const axios = require('axios');

const api = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  }
});

const posts = [
  { content: 'Monday motivation: Start the week strong!', day: 0 },
  { content: 'Tuesday tip: Automate your social media with Publora.', day: 1 },
  { content: 'Wednesday wisdom: Consistency beats perfection.', day: 2 },
  { content: 'Thursday thought: Your audience is waiting.', day: 3 },
  { content: 'Friday feature: Check out our new analytics dashboard!', day: 4 },
  { content: 'Saturday story: How we built Publora.', day: 5 },
  { content: 'Sunday summary: Week in review.', day: 6 },
];

const baseDate = new Date('2026-03-16T10:00:00.000Z');
const results = [];

for (const post of posts) {
  const scheduledTime = new Date(baseDate);
  scheduledTime.setUTCDate(baseDate.getUTCDate() + post.day);

  const { data } = await api.post('/create-post', {
    content: post.content,
    platforms: ['twitter-123', 'linkedin-ABC'],
    scheduledTime: scheduledTime.toISOString()
  });

  results.push({ postGroupId: data.postGroupId, scheduledTime: scheduledTime.toISOString() });
  console.log(`Scheduled: ${post.content.slice(0, 30)}... for ${scheduledTime.toISOString()}`);
}

console.log(`Successfully scheduled ${results.length} posts for the week.`);
```

## Best Practices

1. **Always use UTC times.** Store and transmit all times in ISO 8601 UTC format. Convert to local time only in your UI layer.

2. **Schedule at least 2 minutes ahead.** Although the scheduler runs every minute, scheduling too close to "now" risks the post not being picked up in the current cycle.

3. **Monitor post status.** After scheduling, poll `GET /api/v1/get-post/:postGroupId` to check whether the post moved to `published`, `failed`, or `partially_published`.

4. **Respect free tier limits.** Free trial users can have at most 5 pending (scheduled) posts. Attempting to schedule a 6th returns a `403` error. Upgrade your plan or wait for existing posts to publish before scheduling more.

5. **Use drafts for content approval workflows.** Create posts as drafts, pass them through your review process, and only set `scheduledTime` once approved.

6. **Spread out scheduling times.** Avoid scheduling dozens of posts for the exact same minute. Stagger them by a few minutes to ensure smooth processing.

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| `400 "Invalid scheduled time"` | The `scheduledTime` is in the past or not valid ISO 8601 | Ensure the time is in the future and formatted as `YYYY-MM-DDTHH:mm:ss.sssZ` |
| `403 "Free plan limit reached"` | More than 5 pending posts on the free trial | Wait for existing posts to publish, delete some, or upgrade your plan |
| Post stuck in `scheduled` status | The scheduled time has not arrived yet | Check the `scheduledTime` field -- the scheduler runs every minute |
| `400 "Cannot update published or failed posts"` | Trying to reschedule a post that already published or failed | Only `draft` and `scheduled` posts can be updated |
| Post shows `partially_published` | Some platforms succeeded, others failed | Check the individual platform post statuses within the post group for details |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
