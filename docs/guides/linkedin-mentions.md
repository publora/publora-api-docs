# How to Mention Users and Companies on LinkedIn with Publora

Create LinkedIn posts that @mention people and organizations using the Publora API. Mentions become clickable links and trigger notifications.

## Overview

Publora supports @mentioning both LinkedIn members (people) and organizations (companies) in your posts. When published, mentions render as clickable profile links. The same syntax works in [comments](linkedin-comments.md) and in [reshare commentary](../endpoints/linkedin-reshare.md).

## Prerequisites

- Publora API key
- LinkedIn account connected via Publora dashboard
- LinkedIn URN of the person or company to mention (see [Finding LinkedIn URNs](#finding-linkedin-urns))

> **Important:** Person mentions must use LinkedIn's **native member id** — the short id LinkedIn returns to Publora (e.g. `Dk968RHxiO`). The long `ACoAA…` ids you see in linkedin.com profile URLs and page source are **web-profile ids and do not work**: Publora rejects them with a `400` error. See [Finding LinkedIn URNs](#finding-linkedin-urns) and [Troubleshooting](#troubleshooting).

## Mention Syntax

Use this format in your post content:

```
@{urn:li:person:MEMBER_ID|Display Name}       # Mention a person
@{urn:li:organization:ORG_ID|Company Name}    # Mention an organization
```

## Mention a Person

### JavaScript Example

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Great insights from @{urn:li:person:Dk968RHxiO|Serge Bulaev} on building APIs!',
    platforms: ['linkedin-ABC123']
  })
});
```

**Result on LinkedIn:** `Great insights from @Serge Bulaev on building APIs!`

The mention becomes a clickable link to Serge's profile, and he receives a notification.

### Python Example

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Learned so much from @{urn:li:person:Dk968RHxiO|Serge Bulaev} today!',
        'platforms': ['linkedin-ABC123']
    }
)
```

### cURL Example

```bash
curl -X POST https://api.publora.com/api/v1/create-post \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "content": "Thanks @{urn:li:person:Dk968RHxiO|Serge Bulaev} for the collaboration!",
    "platforms": ["linkedin-ABC123"]
  }'
```

## Mention a Company

### JavaScript Example

```javascript
const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content: 'Excited to partner with @{urn:li:organization:107107343|Creative Content Crafts Inc}!',
    platforms: ['linkedin-ABC123']
  })
});
```

**Result on LinkedIn:** `Excited to partner with @Creative Content Crafts Inc!`

### Python Example

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/create-post',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'content': 'Proud to work with @{urn:li:organization:107107343|Creative Content Crafts Inc}!',
        'platforms': ['linkedin-ABC123']
    }
)
```

## Multiple Mentions

You can mention multiple people and companies in the same post:

```javascript
const content = `
Thrilled to announce our partnership!

Thanks to @{urn:li:person:Dk968RHxiO|Serge Bulaev} and the team at
@{urn:li:organization:107107343|Creative Content Crafts Inc} for making this happen.

Looking forward to building great things together!
`;

const response = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    content,
    platforms: ['linkedin-ABC123']
  })
});
```

## Finding LinkedIn URNs

### Person URN

Format: `urn:li:person:{member_id}`

LinkedIn has **two different id systems** for members, and only one of them works in mentions:

| Id form | Example | Works in mentions? |
|---------|---------|--------------------|
| **Native member id** (app-scoped, returned by LinkedIn's API to Publora) | `Dk968RHxiO` | Yes |
| Web-profile id (from linkedin.com URLs / page source) | `ACoAABcD1234EfG` | **No — rejected with 400** |

Where to get native member ids:

- **Your own connected accounts:** the [platform-connections](../endpoints/platform-connections.md) endpoint returns each LinkedIn connection's `platformId` (e.g. `linkedin-Dk968RHxiO`). Strip the `linkedin-` prefix — the remainder is that member's native id.
- **Actor ids returned by Publora's LinkedIn APIs:** endpoints such as [comments](../endpoints/linkedin-comments.md), [reactions](../endpoints/linkedin-reactions.md), and [feed retrieval](../endpoints/linkedin-feed-retrieval.md) return `urn:li:person:…` actor URNs in their responses. The id inside those URNs is the native form and can be used directly.

> **Honest limitation:** There is **no public API to look up an arbitrary member's native id.** You can reliably mention people whose ids you obtained through Publora or LinkedIn API surfaces (your connections, actors on posts/comments you retrieved) — **not** people you found via a linkedin.com profile URL. The `ACoAA…` id visible in profile URLs and page source cannot be converted to a native id.

### Organization URN

Found in the company page URL or via LinkedIn's API. Format: `urn:li:organization:{numeric_id}`. Organization IDs are numeric (e.g. `107107343`). The numeric admin id shown in your company page URL (`linkedin.com/company/107107343/admin/`) is the correct id.

## Critical: Name Matching Requirements

The display name is matched **case-sensitively** against the profile or page name. The rules differ for people and companies:

**For people:** the display text may be the **first name, last name, or full name** — exactly as it appears on the profile. Any **extra text breaks the mention**: it silently degrades to plain text, with no link and no notification.

| Status | Example |
|--------|---------|
| Correct | `@{urn:li:person:Dk968RHxiO\|Daria Bulaeva}` (full name) |
| Correct | `@{urn:li:person:Dk968RHxiO\|Daria}` (first name only) |
| Wrong | `@{urn:li:person:Dk968RHxiO\|Daria Bulaeva, Ph.D.}` (extra text — plain text, no link) |
| Wrong | `@{urn:li:person:Dk968RHxiO\|daria bulaeva}` (wrong case) |

Never append degrees, suffixes, emojis, or punctuation to a person's name.

**For companies:** use the **exact registered page name** including suffixes like "Inc", "LLC", "Ltd", etc.

| Status | Example |
|--------|---------|
| Correct | `@{urn:li:organization:98765432\|Acme Corp Inc}` |
| Wrong | `@{urn:li:organization:98765432\|Acme Corp}` (missing "Inc") |

**If the name doesn't match:** the mention appears as plain text instead of a clickable link. LinkedIn does not return an error for this — verify the published post.

## Common Mistakes

1. **Web-profile id:** `ACoAABcD1234EfG` instead of the native id `Dk968RHxiO` — rejected with a `400` error
2. **Extra text after a person's name:** `Daria Bulaeva, Ph.D.` instead of `Daria Bulaeva` — silently degrades to plain text
3. **Wrong name case:** `john smith` instead of `John Smith`
4. **Missing company suffix:** `Acme Corp` instead of `Acme Corp Inc`
5. **Extra spaces:** `John  Smith` (double space)
6. **Wrong URN type:** Using `urn:li:member:` instead of `urn:li:person:`

## Troubleshooting

### 400: "LinkedIn cannot render mentions that use web-profile ids"

```
LinkedIn cannot render mentions that use web-profile ids (urn:li:person:ACoAA…).
Use the native member id LinkedIn returns to Publora. External connection identifiers
are namespaced (for example linkedin-Dk968RHxiO), so remove the leading linkedin-
before building the person URN; actor ids from Publora's LinkedIn APIs are already
raw. IDs copied from linkedin.com profile URLs will not work.
```

Publora returns this `400` at intake (create-post, update-post, linkedin-comments, linkedin-reshare) when a person mention uses an `ACoAA…` web-profile id. Replace it with the native member id (see [Finding LinkedIn URNs](#finding-linkedin-urns)). Before this validation existed, LinkedIn itself rejected such posts with `400 INVALID_CONTENT` ("Person URN ID in commentary field is invalid"), and comments failed with an opaque `403`.

### Post published, but the whole text shows literal `@{...}` markup with backslashes

LinkedIn sometimes accepts a post containing a web-profile (`ACoAA…`) person id but fails to parse the mention — the entire commentary then renders as escaped literal text (visible `@{urn:li:person:…|Name}` and backslashes). This is another symptom of the same root cause: switch to the native member id.

### Mention shows as plain text (no link, no notification)

The id was valid but the display name didn't match. Check the [name matching rules](#critical-name-matching-requirements): case must match, and for people the text must be exactly the first name, last name, or full name — nothing more.

## Verifying Mentions Work

After posting, check your LinkedIn post to ensure:
1. The mention appears as a clickable blue link
2. Clicking it navigates to the correct profile/company page
3. The mentioned person/company received a notification

## Related Guides

- [LinkedIn Comments](linkedin-comments.md) - Comment on posts
- [LinkedIn Reshare](../endpoints/linkedin-reshare.md) - Reshare posts with mention-capable commentary
- [LinkedIn Reactions](linkedin-reactions.md) - Like and react to posts
- [LinkedIn Platform Guide](../platforms/linkedin.md) - Complete LinkedIn API reference
