# Account Context

Read the plan, available features, publishing quotas, schedule horizon and allowed platforms of the account behind your API key. Call it before a batch of posts and again after a quota or feature error, so your integration knows how many platform posts it can still publish and how far ahead it can schedule.

The response is a read-only, **advisory** snapshot: it reserves nothing, and every write is still checked against the limits when it is made. The MCP tool [`account_context`](../mcp/tools-reference.md#account_context) returns the same object.

## Endpoint

```
GET https://api.publora.com/api/v1/account-context
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace keys only). Returns that user's context instead of the key owner's. |

## Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `scheduledTime` | string | No | ISO 8601 date-time with `Z` or an explicit offset, e.g. `2026-11-03T09:00:00Z` or `2026-11-03T10:00:00+01:00`. Selects the monthly quota period of a planned batch. Omit it for the current period. |

> **`scheduledTime` only selects the quota period.** A past time resolves to the current period, and `effectiveQuotaTime` in the response shows the date that was used. It does not make a past schedule valid: `create-post` and `update-post` still apply their own rules to the time you send.

## Response

```json
{
  "success": true,
  "context": {
    "plane": "personal",
    "actorUserId": "665f1c2ab3e4d5f6a7b8c9d0",
    "snapshotAt": "2026-10-02T08:37:17.484Z",
    "serverTime": "2026-10-02T08:37:17.484Z",
    "effectiveQuotaTime": "2026-10-02T08:37:17.484Z",
    "plan": { "key": "starter", "name": "Starter", "source": "starter" },
    "features": {
      "publishing": true,
      "apiAccess": true,
      "mcpAccess": true,
      "analytics": false,
      "mentionables": false
    },
    "quotas": {
      "monthlyPosts": {
        "metric": "posts.platform_monthly",
        "scope": "account",
        "usageScope": "legacy_workspace",
        "unit": "platform_post",
        "limit": 15,
        "used": 4,
        "remaining": 11,
        "periodStart": "2026-10-01T00:00:00.000Z",
        "periodEnd": "2026-11-01T00:00:00.000Z",
        "resetAt": "2026-11-01T00:00:00.000Z",
        "periodTimeZone": "UTC",
        "connections": []
      },
      "scheduledPosts": {
        "metric": "posts.scheduled_active",
        "scope": "account",
        "usageScope": "legacy_workspace",
        "unit": "platform_post",
        "limit": 3,
        "used": 1,
        "remaining": 2,
        "resetAt": null
      },
      "scheduleHorizon": {
        "metric": "posts.schedule_horizon_days",
        "unit": "day",
        "limit": 7,
        "maxScheduledDate": "2026-10-09T23:59:59.999Z",
        "resetAt": null
      },
      "connections": {
        "metric": "connections.total",
        "scope": "account",
        "usageScope": "legacy_workspace",
        "unit": "channel",
        "limit": 3,
        "used": 2,
        "remaining": 1
      }
    },
    "allowedPlatforms": ["threads", "bluesky", "mastodon", "tiktok", "instagram", "youtube", "linkedin", "facebook", "pinterest", "telegram"],
    "advisory": true
  }
}
```

This Starter account has 11 of its 15 platform posts left this month (the count starts over at `resetAt`), room for 2 more scheduled platform posts, a schedule horizon that ends on 2026-10-09, and no X (`twitter`) in `allowedPlatforms`.

### Response fields (inside `context`)

| Field | Type | Description |
|-------|------|-------------|
| `plane` / `actorUserId` | string | Always `personal`, and the ID of the user the snapshot describes |
| `snapshotAt` / `serverTime` | string | When the snapshot was taken |
| `effectiveQuotaTime` | string | The date used to select the monthly quota period |
| `plan` | object | `key`, `name` and `source` of the plan in effect |
| `features` | object | `true` or `false` for `publishing`, `apiAccess`, `mcpAccess`, `analytics` (needed by [Mastodon and Bluesky Statistics](platform-statistics.md)) and `mentionables` (needed by [LinkedIn Mentionables](linkedin-mentionables.md)) |
| `quotas.monthlyPosts` | object | Platform posts in the monthly period: `limit`, `used`, `remaining`, `periodStart`, `periodEnd`, `resetAt` (the end of the period), `periodTimeZone`, and `connections[]` for per-connection limits |
| `quotas.scheduledPosts` | object | Platform posts waiting to publish at a future time, against the plan's queue limit: `limit`, `used`, `remaining`. Posts sent with [`publishNow`](create-post.md#publish-now) are not counted. `resetAt` is `null`. |
| `quotas.scheduleHorizon` | object | `limit` in days and `maxScheduledDate`, the latest time a post can be scheduled (`null` when there is no horizon). `resetAt` is `null`. |
| `quotas.connections` | object | Connected channels: `limit`, `used`, `remaining` |
| `metric`, `scope`, `usageScope`, `unit` | string | Identify each quota. `unit` is `platform_post`, `day` or `channel`; `scope` is `account` (one shared limit) or `connection` (a limit for each connection) |
| `allowedPlatforms` | string[]/null | Platform types the plan allows; `null` means all platforms |
| `advisory` | boolean | Always `true`: the snapshot reserves nothing |

> **Units are platform posts.** A post to three platforms uses three from `monthlyPosts`, and three from `scheduledPosts` while it waits to publish. `limit: null` means unlimited.

> **Per-connection limits.** When `quotas.monthlyPosts.scope` is `"connection"`, the limit applies to each connection separately. Read `quotas.monthlyPosts.connections[]`: each entry has `platformSelection` (the connection ID you pass to `create-post`), `used`, `limit` and `remaining`. The aggregate `remaining` is `null` in that case, which does not mean unlimited.

## Examples

### cURL

```bash
curl https://api.publora.com/api/v1/account-context \
  -H "x-publora-key: YOUR_API_KEY"

# Quota period of a batch planned for November
curl "https://api.publora.com/api/v1/account-context?scheduledTime=2026-11-03T09:00:00Z" \
  -H "x-publora-key: YOUR_API_KEY"
```

### JavaScript (fetch)

```javascript
const res = await fetch('https://api.publora.com/api/v1/account-context', {
  headers: { 'x-publora-key': 'YOUR_API_KEY' }
});
const { context } = await res.json();
const { remaining, limit } = context.quotas.monthlyPosts;
console.log(limit === null ? 'Unlimited posts' : `${remaining} of ${limit} platform posts left`);
```

### Python (requests)

```python
import requests

res = requests.get(
    'https://api.publora.com/api/v1/account-context',
    headers={'x-publora-key': 'YOUR_API_KEY'},
    params={'scheduledTime': '2026-11-03T09:00:00Z'},
)
context = res.json()['context']
print(context['plan']['name'], context['quotas']['scheduleHorizon']['maxScheduledDate'])
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"scheduledTime must be a valid ISO date-time"` | `code: "INVALID_SCHEDULED_TIME"`, `retryable: false`, `nextAction: "CHANGE_SCHEDULE"`. The value has no `Z` or offset, or is an impossible date such as `2026-02-30`. |
| 401 | `"API key is required"` / `"Invalid API key"` | Missing or invalid `x-publora-key` |
| 403 | `"API access is not enabled for this account"` | The plan has no API access |
| 500 | `"Failed to read account context"` | Internal server error; retry later |

## Notes

- **A snapshot, not a reservation.** Every write is checked again when it is made, so a write can still hit a limit if the quota was used in the meantime.
- **Same numbers as enforcement.** The quotas use the same counting as `create-post` and `update-post`, including the [Publish Now](create-post.md#publish-now) exemption from the scheduled-post queue.
