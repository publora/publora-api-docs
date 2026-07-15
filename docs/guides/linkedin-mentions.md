# How to Mention Users and Companies on LinkedIn with Publora

Create LinkedIn posts that @mention people and organizations using the Publora API. Mentions become clickable links and trigger notifications.

## Overview

Publora supports @mentioning both LinkedIn members (people) and organizations (companies) in your posts. When published, mentions render as clickable profile links.

## Prerequisites

- Publora API key
- LinkedIn account connected via Publora dashboard
- LinkedIn URN of the person or company to mention (see [Finding LinkedIn URNs](#finding-linkedin-urns))

> **Important:** You must use a valid LinkedIn URN ID. Invalid or made-up IDs will cause a `400` error: `"Person URN ID in commentary field is invalid."` LinkedIn person IDs are typically long alphanumeric strings (e.g. `ACoAABcD1234EfG`) — short numeric IDs like `123` or `456` will not work.

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
    content: 'Great insights from @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} on building APIs!',
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
        'content': 'Learned so much from @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} today!',
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
    "content": "Thanks @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} for the collaboration!",
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

Thanks to @{urn:li:person:ACoAABcD1234EfG|Serge Bulaev} and the team at
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
The member ID from LinkedIn's API. Format: `urn:li:person:{member_id}`

LinkedIn member IDs are typically long alphanumeric strings (e.g. `ACoAABcD1234EfG`). You can find them via:
- LinkedIn's API (`/me` endpoint returns your own URN)
- Browser developer tools: inspect network requests when viewing someone's profile
- The LinkedIn profile URL sometimes contains a numeric ID, but this is **not** the same as the person URN ID

> **Warning:** Using an invalid or made-up URN ID will cause LinkedIn to reject the entire post with a `400` error. Always verify the URN is correct before posting.

### Organization URN
Found in the company page URL or via LinkedIn's API. Format: `urn:li:organization:{numeric_id}`. Organization IDs are typically 8-digit numbers (e.g. `98765432`).

## Critical: Name Matching Requirements

**The display name MUST exactly match the name on LinkedIn** (case-sensitive):

| Status | Example |
|--------|---------|
| Correct | `@{urn:li:organization:98765432\|Acme Corp Inc}` |
| Wrong | `@{urn:li:organization:98765432\|Acme Corp}` (missing "Inc") |
| Correct | `@{urn:li:person:ACoAADeFgHi5678\|John Smith}` |
| Wrong | `@{urn:li:person:ACoAADeFgHi5678\|john smith}` (wrong case) |

**For companies:** Use the **exact registered company name** including suffixes like "Inc", "LLC", "Ltd", etc.

**If the name doesn't match:** The mention will appear as plain text instead of a clickable link.

## Common Mistakes

1. **Wrong name case:** `john smith` instead of `John Smith`
2. **Missing company suffix:** `Acme Corp` instead of `Acme Corp Inc`
3. **Extra spaces:** `John  Smith` (double space)
4. **Wrong URN type:** Using `urn:li:member:` instead of `urn:li:person:`

## Verifying Mentions Work

After posting, check your LinkedIn post to ensure:
1. The mention appears as a clickable blue link
2. Clicking it navigates to the correct profile/company page
3. The mentioned person/company received a notification

## Related Guides

- [LinkedIn Comments](linkedin-comments.md) - Comment on posts
- [LinkedIn Reactions](linkedin-reactions.md) - Like and react to posts
- [LinkedIn Platform Guide](../platforms/linkedin.md) - Complete LinkedIn API reference
