# Scheduling Posts

This guide covers how to schedule social media posts through the Publora API, including creating drafts, scheduling for specific times, and managing post lifecycles.

## How It Works

Every post in Publora follows a lifecycle defined by its status:

```
status field:            draft --> scheduled --> published
                                            \-> failed
                                            \-> partially_published

processingStatus field:  pending --> processing --> finished
```

- **draft** -- A post saved without a `scheduledTime`. It will not be published until you update it with a valid time.
- **scheduled** -- A post with a future `scheduledTime`. The Publora scheduler checks for due posts every minute.
- **published** -- All platform posts succeeded.
- **failed** -- All platform posts failed.
- **partially_published** -- Some platform posts succeeded while others failed.

> **Note:** `processing` is **not** a value of the `status` field. Processing state is tracked separately via the `processingStatus` field, which transitions through `pending` -> `processing` -> `finished`. The `status` field only contains: `draft`, `scheduled`, `published`, `failed`, or `partially_published`.

### Key Rules

- `scheduledTime` must be in **ISO 8601 UTC format** (e.g., `2026-03-15T14:30:00.000Z`). An invalid format returns a `400 "Invalid scheduled time format"` error.
- A `scheduledTime` that is in the past or exactly equal to `now` (`scheduledTime <= now`) is **never silently accepted**. See [Past scheduled times](#past-scheduled-times) below -- the API either clamps it and warns you, or rejects it.
- **Starter plan** users can have a maximum of **3 pending (scheduled) posts** at any time. This counts individual platform-level posts, not post groups. A single post group targeting 3 platforms counts as 3 scheduled posts toward this limit. Pro and Premium plan `scheduledPosts` limits are per-connection (100 and 500 respectively), not account-wide like Starter's limit of 3.
- **Schedule horizon:** Starter plan users can schedule up to **7 days** in advance. Pro and Premium plans have no horizon limit.
- **Monthly post limits:** Starter allows **15 posts/month** (account-wide), Pro allows **100 posts/month per connection**, Premium allows **500 posts/month per connection**. Note: scheduled posts are counted per-platform post, not per post group. A single post group targeting 3 platforms counts as 3 posts toward your limit.
- **Starter plan can post to any of the 10 platforms** (no platform restriction).
- The scheduler runs every minute, so posts are published within roughly 60 seconds of their scheduled time.

### Past scheduled times

Previously, a `scheduledTime` in the past was silently rewritten to the current time -- so "post at 09:00" computed a few seconds late, or a stale retry of an old request, published **immediately** with no signal. That is fixed: the API now always tells you.

| How far in the past | Today | From **2026-08-25** |
|---|---|---|
| Less than 5 minutes (clock skew) | Clamped to server time + `warnings` in the 2xx body | Unchanged -- still clamped + warned |
| 5 minutes or more | Clamped to server time + `warnings` in the 2xx body | **`400 SCHEDULED_TIME_IN_PAST`** |

The 5-minute tolerance for clock skew is permanent. Only the 5-minutes-or-more case changes.

**Warning (2xx)** -- the post *was* created; `effective` is the time actually stored:

```json
{
  "success": true,
  "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d",
  "warnings": [
    {
      "code": "SCHEDULED_TIME_COERCED",
      "message": "Requested scheduled time 2026-03-15T14:29:58.000Z was in the past and was changed to server time 2026-03-15T14:30:02.104Z.",
      "requested": "2026-03-15T14:29:58.000Z",
      "effective": "2026-03-15T14:30:02.104Z"
    }
  ]
}
```

**Rejection (400)** -- once the change lands, nothing is created:

```json
{
  "error": "Scheduled time is in the past. Server time is 2026-03-15T14:30:02.104Z UTC.",
  "code": "SCHEDULED_TIME_IN_PAST",
  "serverTime": "2026-03-15T14:30:02.104Z"
}
```

`serverTime` is the authoritative UTC clock -- compare it against your own to detect drift, and retry with a `scheduledTime` after it.

**What to do now**

- Treat any `warnings` entry with `code: "SCHEDULED_TIME_COERCED"` as an error in the making -- it becomes a `400` on 2026-08-25.
- Build times from UTC (`new Date().toISOString()`, `datetime.now(timezone.utc)`), not local wall-clock.
- Add a small buffer (30-60s) when scheduling "now" to absorb network and clock skew.
- To confirm what the server kept, read the post back -- [`GET /get-post`](../endpoints/get-post.md) returns the stored **effective** `scheduledTime`, not what you asked for.

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
// Response: { success: true, postGroupId: "664f1a2b3c4d5e6f7a8b9c0d" }
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
# Response: { "success": true, "postGroupId": "664f1a2b3c4d5e6f7a8b9c0d" }
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

Sometimes you want to prepare content first and schedule it at a later time. Omit `scheduledTime` to create a draft, then update it with a `PUT` request. When scheduling a draft, you must send **both** `scheduledTime` and `status: "scheduled"` in the update request.

> **Note:** Platform availability checks (`assertPlatformsAllowed`) run at `create-post` time for ALL posts, regardless of status. Other plan limits (monthly post count, scheduled post count, schedule horizon) run when a post transitions to `scheduled` status.

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
      status: 'scheduled',
      scheduledTime: '2026-03-20T09:00:00.000Z'
    })
  }
);

const scheduled = await scheduleResponse.json();
// Response: { message: "Post updated successfully", postGroup: { _id: "...", status: "scheduled", scheduledTime: "2026-03-20T09:00:00.000Z" } }
console.log('Now scheduled:', scheduled.postGroup.status); // "scheduled"
```

> **Note:** The `update-post` response includes `message: "Post updated successfully"` and a `postGroup` object containing `_id`, `status`, and optionally `scheduledTime` (present when the post is scheduled).

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

# Step 2: Schedule the draft (both status and scheduledTime are required)
schedule_response = requests.put(
    f"{API_URL}/update-post/{draft['postGroupId']}",
    headers=HEADERS,
    json={
        'status': 'scheduled',
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
    "status": "scheduled",
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

// Step 2: Schedule the draft (both status and scheduledTime are required)
const { data: scheduled } = await api.put(`/update-post/${draft.postGroupId}`, {
  status: 'scheduled',
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

4. **Respect Starter plan limits.** Starter plan users can have at most 3 pending (scheduled) posts and 15 posts per month. Exceeding these limits returns a `403` error with a structured `LimitExceededError` response. Upgrade your plan or wait for existing posts to publish before scheduling more.

5. **Use drafts for content approval workflows.** Create posts as drafts, pass them through your review process, and only set `scheduledTime` once approved.

6. **Spread out scheduling times.** Avoid scheduling dozens of posts for the exact same minute. Stagger them by a few minutes to ensure smooth processing.

## Common Issues

| Problem | Cause | Solution |
|---|---|---|
| `400 "Invalid scheduled time format"` | The `scheduledTime` is not valid ISO 8601 | Ensure the time is formatted as `YYYY-MM-DDTHH:mm:ss.sssZ`. Note: past times are silently rounded up to `now` rather than rejected |
| `403` limit exceeded | More than 3 pending posts on the Starter plan, or monthly limit reached | Wait for existing posts to publish, delete some, or upgrade your plan. The response includes a structured `LimitExceededError` with `metric`, `limit`, `used`, and `remaining` fields |
| Post stuck in `scheduled` status | The scheduled time has not arrived yet | Check the `scheduledTime` field -- the scheduler runs every minute |
| `400 "Cannot update post: post is currently in {status} status"` | Trying to reschedule a post that is in a non-editable status | Only `draft` and `scheduled` posts can be updated |
| Post shows `partially_published` | Some platforms succeeded, others failed | Check the individual platform post statuses within the post group for details |


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
