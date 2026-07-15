# Platform Limits

Return Publora's per-platform posting limits (character counts, image / video / document / GIF / thumbnail rules, and platform requirements) as live JSON. These are the **API-specific** limits Publora validates against before scheduling — fetch them to drive client-side validation instead of hard-coding values. For the human-readable tables and the API-vs-native-app differences, see the [Platform Limits guide](../guides/platform-limits.md).

The limits come from the shared `@publora/platform-limits` package, so this endpoint always reflects the same rules the scheduler enforces.

## Endpoint

```
GET https://api.publora.com/api/v1/platform-limits
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |

No query parameters or request body.

## Response

The JSON below is abridged: only `twitter` is shown under `platforms`. The live response includes an object for every one of the 11 `supportedPlatforms` keys.

```json
{
  "success": true,
  "schemaVersion": 1,
  "source": "@publora/platform-limits",
  "packageVersion": "1.0.0",
  "limitsLastUpdated": "2026-03-11",
  "supportedPlatforms": ["twitter", "instagram", "threads", "tiktok", "linkedin", "youtube", "facebook", "mastodon", "bluesky", "telegram", "pinterest"],
  "platforms": {
    "twitter": {
      "platform": "twitter",
      "displayName": "Twitter/X",
      "characters": { "standard": 280, "premium": 25000 },
      "images": { "supported": true, "maxSizeBytes": 5242880, "maxCount": 4, "formats": ["image/jpeg", "image/png", "image/gif", "image/webp"] },
      "videos": { "supported": true, "maxDurationSeconds": 140, "minDurationSeconds": 0.5, "maxSizeBytes": 536870912, "maxCount": 1, "formats": ["video/mp4", "video/quicktime"], "aspectRatioRange": { "min": 0.3333333333333333, "max": 3 } },
      "requirements": { "requiresMedia": false, "requiresVideo": false, "supportsTextOnly": true, "supportsThreading": true },
      "documents": null,
      "gifs": null,
      "thumbnails": null
    }
  }
}
```

| Field | Description |
|-------|-------------|
| `success` | `true` when the limits were returned. |
| `schemaVersion` | Response schema version (currently `1`); bumped on breaking shape changes. |
| `source` | Always `"@publora/platform-limits"` — the package the values come from. |
| `packageVersion` | Version of that package (e.g. `"1.0.0"`). |
| `limitsLastUpdated` | ISO date the limit values were last revised. |
| `supportedPlatforms` | Array of platform keys included in `platforms`. |
| `platforms` | Object keyed by platform. Each value carries `platform`, `displayName`, `characters`, `images`, `videos`, `requirements`, plus `documents`, `gifs`, and `thumbnails` (each `null` when the platform has no distinct spec). |

> **Pinterest is present** in `supportedPlatforms` and `platforms` as an internal reference key, but Pinterest is **not currently available** for posting — treat the other 10 platforms as active.

## Usage

### cURL

```bash
curl https://api.publora.com/api/v1/platform-limits \
  -H "x-publora-key: $PUBLORA_API_KEY"
```

### JavaScript (fetch)

```javascript
const res = await fetch('https://api.publora.com/api/v1/platform-limits', {
  headers: { 'x-publora-key': process.env.PUBLORA_API_KEY },
});
const { platforms } = await res.json();
const twitterMax = platforms.twitter.characters.standard; // 280
```

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| `401` | `Invalid API key` | Missing or invalid `x-publora-key`. |

## Notes

- **Cache friendly.** Limits change rarely — cache the response and refresh when `limitsLastUpdated` changes rather than calling per request.
- **API limits, not native-app limits.** These reflect what the third-party APIs accept, which is often stricter than the platforms' own apps. See the [guide](../guides/platform-limits.md) for the differences.
- **Units.** `maxSizeBytes` is in bytes; durations are in seconds.
