# Mastodon and Bluesky Statistics

Engagement counters for published Mastodon and Bluesky posts, and follower counts for the connected accounts. Values are read from the platform when you ask for them and cached for about 2 hours.

Two endpoints:

```
POST https://api.publora.com/api/v1/post-statistics
POST https://api.publora.com/api/v1/profile-statistics
```

Both require a plan that includes analytics (Pro or Premium). A Starter key receives `403 ANALYTICS_PLAN_REQUIRED`.

> LinkedIn analytics live on separate endpoints with a different contract — see [LinkedIn Statistics](https://docs.publora.com/endpoints/linkedin-statistics).

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

## What can be queried

- **Only posts Publora published for you.** A `postedId` that does not belong to one of your own published posts is answered `null` without contacting the platform. That includes posts published from another account, invented IDs, and posts Publora published before it began storing platform IDs for these two platforms (2026-09-07) — those older posts have no `postedId` and cannot be queried.
- **Threads: the root post only.** `GET /get-post` returns one `postedId` per platform post, which for a thread is the first part. The remaining parts are not addressable.
- Take `postedId` from [`GET /get-post`](https://docs.publora.com/endpoints/get-post) or [`GET /list-posts`](https://docs.publora.com/endpoints/list-posts), and `platformId` from [`GET /platform-connections`](https://docs.publora.com/endpoints/platform-connections).

## Post Statistics

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `posts` | object[] | Yes | 1–50 entries. Each entry is one post on one connection. |
| `posts[].platform` | string | Yes | `mastodon` or `bluesky` |
| `posts[].platformId` | string | Yes | Connection ID, with or without the platform prefix: `bluesky-did:plc:abc123` or `did:plc:abc123` |
| `posts[].postedId` | string | Yes | Platform post ID: a Bluesky AT-URI (`at://did:plc:.../app.bsky.feed.post/3k...`) or a Mastodon status ID (`117232239423110999`) |

One request may mix both platforms and several connections. Every string is capped at 512 characters.

```json
{
  "posts": [
    {
      "platform": "bluesky",
      "platformId": "bluesky-did:plc:3xcxmi4aiok5zyghylsa4dzw",
      "postedId": "at://did:plc:3xcxmi4aiok5zyghylsa4dzw/app.bsky.feed.post/3muxmedtwxd2k"
    },
    {
      "platform": "mastodon",
      "platformId": "mastodon-110300915972205108",
      "postedId": "117232239423110999"
    }
  ]
}
```

### Response

```json
{
  "success": true,
  "stats": {
    "at://did:plc:3xcxmi4aiok5zyghylsa4dzw/app.bsky.feed.post/3muxmedtwxd2k": {
      "reactions": 42,
      "comments": 3,
      "reposts": 7,
      "quotes": 1,
      "saves": 2,
      "impressions": null,
      "reach": null,
      "clicks": null
    },
    "117232239423110999": null
  }
}
```

| Field | Type | Description |
|-------|------|-------------|
| `stats` | object | Keyed by `postedId`. The value is a metrics object, or `null` when there is no data right now. |
| `rateLimited` | boolean | Present and `true` only when at least one connection was in a rate-limit cooldown |
| `issues` | object | Present only when non-empty. Keyed by connection ID in the `<platform>-<platformId>` form, value is one of the codes below. |

`null` in `stats` is not an error. It means: the post was deleted on the platform, the ID is not one of your published posts, or the connection hit one of the conditions reported in `issues`. Retry later rather than treating it as a permanent zero.

### Metrics

Every key is always present; a metric the platform does not expose is `null`, never `0`.

| Field | Mastodon | Bluesky |
|-------|----------|---------|
| `reactions` | Favourites | Likes |
| `comments` | Replies | Replies |
| `reposts` | Boosts | Reposts |
| `quotes` | Quotes (Mastodon 4.5+, otherwise `null`) | Quote posts |
| `saves` | `null` | Bookmarks |
| `impressions` | `null` | `null` |
| `reach` | `null` | `null` |
| `clicks` | `null` | `null` |

### Connection issues

| Code | Meaning | What to do |
|------|---------|------------|
| `CONNECTION_NOT_FOUND` | No connection of yours matches that `platform` + `platformId` | Re-read `GET /platform-connections` |
| `AUTH_REVOKED` | Mastodon rejected the stored token (HTTP 401) | Reconnect the account; the connection is also flagged in `GET /platform-connections` |
| `FORBIDDEN` | Mastodon refused analytics access (HTTP 403, scope or moderation). Publishing is unaffected. | Reconnect if it persists; otherwise retry later |
| `RATE_LIMITED` | The platform is in a cooldown after answering 429 | Retry later. Mastodon cooldowns are per connection; Bluesky cooldowns apply to the whole platform. |
| `FETCH_FAILED` | A transient failure reaching the platform | Retry later |

## Profile Statistics

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `platform` | string | Yes | `mastodon` or `bluesky` |
| `platformId` | string | Yes | Connection ID, prefix optional |

```json
{
  "platform": "mastodon",
  "platformId": "mastodon-110300915972205108"
}
```

### Response

```json
{
  "success": true,
  "profile": {
    "followers": 1234,
    "following": 321,
    "posts": 987
  },
  "cached": true,
  "fetchedAt": "2026-09-08T09:00:00.000Z"
}
```

| Field | Type | Description |
|-------|------|-------------|
| `profile` | object \| null | `followers`, `following`, `posts`. Each value is a number or `null`. `profile` itself is `null` when the account could not be read. |
| `cached` | boolean | `true` when the answer came from the ~2-hour cache |
| `fetchedAt` | string \| null | ISO 8601 timestamp of the value that was returned |
| `rateLimited` | boolean | Present and `true` only during a cooldown |
| `unavailable` | string | Present only when the connection is marked unavailable: `AUTH_REVOKED` or `FORBIDDEN` |

Unlike post statistics, an unknown connection is a hard error here: `404 <platform> connection not found`.

## Caching and load

- Post and profile values are cached for about 2 hours per connection. Two Publora users who connected the same account share that cache.
- A post that no longer exists on the platform is remembered as missing for the same period.
- After a platform answers `429`, Publora stops calling it until the reset time it reported (at most one hour) and answers `rateLimited: true` in the meantime.

## Examples

### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/post-statistics', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    posts: [
      {
        platform: 'bluesky',
        platformId: 'bluesky-did:plc:3xcxmi4aiok5zyghylsa4dzw',
        postedId: 'at://did:plc:3xcxmi4aiok5zyghylsa4dzw/app.bsky.feed.post/3muxmedtwxd2k'
      }
    ]
  })
});
const data = await response.json();

for (const [postedId, metrics] of Object.entries(data.stats)) {
  if (!metrics) {
    console.log(`${postedId}: no data right now`);
    continue;
  }
  console.log(`${postedId}: ${metrics.reactions} likes, ${metrics.comments} replies`);
}
```

### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/post-statistics',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'posts': [
            {
                'platform': 'mastodon',
                'platformId': 'mastodon-110300915972205108',
                'postedId': '117232239423110999'
            }
        ]
    }
)
data = response.json()

for posted_id, metrics in data['stats'].items():
    if metrics is None:
        print(f'{posted_id}: no data right now')
    else:
        print(f"{posted_id}: {metrics['reactions']} favourites, {metrics['reposts']} boosts")

for connection_id, code in data.get('issues', {}).items():
    print(f'{connection_id} needs attention: {code}')
```

### cURL

```bash
curl -X POST https://api.publora.com/api/v1/post-statistics \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "posts": [
      {
        "platform": "bluesky",
        "platformId": "bluesky-did:plc:3xcxmi4aiok5zyghylsa4dzw",
        "postedId": "at://did:plc:3xcxmi4aiok5zyghylsa4dzw/app.bsky.feed.post/3muxmedtwxd2k"
      }
    ]
  }'
```

```bash
curl -X POST https://api.publora.com/api/v1/profile-statistics \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "platform": "mastodon",
    "platformId": "mastodon-110300915972205108"
  }'
```

### Node.js (axios) — stats for a page of published posts

```javascript
const axios = require('axios');

const client = axios.create({
  baseURL: 'https://api.publora.com/api/v1',
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY }
});

async function statsForPublishedPage() {
  const { data: list } = await client.get('/list-posts', {
    params: { status: 'published', limit: 25 }
  });

  const posts = list.posts
    .flatMap((group) => group.posts || [])
    .filter((post) => ['mastodon', 'bluesky'].includes(post.platform) && post.postedId)
    .map((post) => ({
      platform: post.platform,
      platformId: post.platformId,
      postedId: post.postedId
    }));

  if (posts.length === 0) return {};

  // Never send more than 50 in one request.
  const { data } = await client.post('/post-statistics', { posts: posts.slice(0, 50) });
  if (data.issues) console.warn('Connections needing attention:', data.issues);
  return data.stats;
}
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"posts array is required"` | `posts` missing, not an array, or empty |
| 400 | `"Maximum 50 posts per request"` | More than 50 entries in `posts` |
| 400 | `"Each post must have string platform, platformId and postedId"` | An entry is missing a field, has a non-string value, or exceeds 512 characters |
| 400 | `"platform and platformId are required"` | Profile statistics called without both fields |
| 400 | `"Unsupported platform \"x\". Supported: mastodon, bluesky"` | `platform` is neither `mastodon` nor `bluesky` |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 403 | `"Analytics requires a Pro or Premium plan"` | Code `ANALYTICS_PLAN_REQUIRED`; the plan has no analytics feature |
| 404 | `"mastodon connection not found"` | Profile statistics only — no connection of yours matches. Post statistics report this as `issues[…] = "CONNECTION_NOT_FOUND"` with `200`. |
| 504 | `"Analytics request timed out"` | Code `ANALYTICS_REQUEST_TIMEOUT`; the request exceeded its 90-second budget. Retry with fewer posts. |
| 500 | `"Failed to fetch post statistics"` | Unexpected server error |
| 500 | `"Failed to fetch profile statistics"` | Unexpected server error |

Per-connection failures are **not** errors: the response stays `200` and reports them in `issues` (post statistics) or `unavailable` / `rateLimited` (profile statistics).

## MCP

The same two operations are available over the [MCP server](https://docs.publora.com/guides/mcp-server) as `post_stats` and `profile_stats` — see the [MCP Tools Reference](https://docs.publora.com/mcp/tools-reference).

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
