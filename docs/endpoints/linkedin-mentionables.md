# LinkedIn Mentionables

List the LinkedIn members you can @mention — a per-user directory of people whose **native member ids** Publora has captured from engagement on your connected company pages. Each entry includes a ready-to-paste `mention` token for [create-post](create-post.md) content, [comment](linkedin-comments.md) messages, or [reshare commentary](linkedin-reshare.md).

Why this exists: person mentions require LinkedIn's native member id (e.g. `Dk968RHxiO`), which cannot be derived from a linkedin.com profile URL. This directory is where Publora surfaces every native id it has seen for you. See the [LinkedIn Mentions Guide](../guides/linkedin-mentions.md) for the full id explanation and name-matching rules.

> **Paid plans only.** Free plans receive a `403 UPGRADE_REQUIRED` error (see [Errors](#errors)).

## List Mentionable People

```
GET https://api.publora.com/api/v1/linkedin-mentionables
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |

### Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `q` | string | No | Case-insensitive substring filter on the person's name (regex characters are treated literally). Omit to list everyone. |
| `limit` | number | No | Maximum number of entries to return, `1`–`100`. Default `25`. |

Results are sorted by `lastSeenAt` descending — the most recently engaged people first.

### Response (HTTP 200)

```json
{
  "success": true,
  "people": [
    {
      "personId": "Dk968RHxiO",
      "name": "Daria Bulaeva",
      "profileUrl": "",
      "profilePicture": "",
      "source": "comment",
      "lastSeenAt": "2026-07-16T10:00:00.000Z",
      "mention": "@{urn:li:person:Dk968RHxiO|Daria Bulaeva}"
    }
  ]
}
```

| Field | Type | Description |
|-------|------|-------------|
| `personId` | string | The person's **native member id** — the only id form that works in mentions. |
| `name` | string | The display name as captured from LinkedIn. |
| `profileUrl` | string | Currently always an empty string (reserved for future use). |
| `profilePicture` | string | Currently always an empty string (reserved for future use). |
| `source` | string | How the person was last captured: `comment` or `reaction`. The most recent engagement wins. |
| `lastSeenAt` | string | ISO 8601 timestamp of the most recent captured engagement. |
| `mention` | string \| null | Ready-to-paste mention token (`@{urn:li:person:ID\|Name}`) for post content or comment messages. `null` when no usable name is stored. Names containing `\|` or `}` are sanitized in the token. |

> **Note:** The `mention` token uses the person's full captured name. LinkedIn matches display names strictly — if you edit the name part, follow the [name matching rules](../guides/linkedin-mentions.md#critical-name-matching-requirements) (first name, last name, or full name; exact case; nothing extra).

### Examples

#### JavaScript (fetch)

```javascript
const params = new URLSearchParams({ q: 'daria', limit: '10' });
const response = await fetch(
  `https://api.publora.com/api/v1/linkedin-mentionables?${params}`,
  { headers: { 'x-publora-key': 'YOUR_API_KEY' } }
);
const data = await response.json();

for (const person of data.people) {
  if (person.mention) {
    console.log(`${person.name}: ${person.mention}`);
  }
}
```

#### cURL

```bash
curl "https://api.publora.com/api/v1/linkedin-mentionables?q=daria&limit=10" \
  -H "x-publora-key: YOUR_API_KEY"
```

## How the Directory Fills

The directory is populated **automatically** — there is no endpoint to add people manually:

- When comments or reactions are read on posts of your connected **company pages** (via the Publora app's engagement views or the [feed-retrieval API](linkedin-feed-retrieval.md)), LinkedIn returns each engaging member's native app-scoped person id, and Publora persists it in your directory.
- No extra LinkedIn API calls are made — the ids are captured as a side effect of reads you already perform.
- Engagement on **personal-profile posts is not capturable** — LinkedIn restricts the required scope. Only company-page engagement feeds the directory.
- Placeholder/unresolved actors (entries LinkedIn returns without a resolvable member) are skipped.

There is **no way to add an arbitrary person by profile URL** — the `ACoAA…` web-profile ids visible on linkedin.com cannot be converted to native ids. See [Finding LinkedIn URNs](../guides/linkedin-mentions.md#finding-linkedin-urns) for the full explanation of native vs web-profile ids.

## MCP Tool

The same directory is available to MCP clients as the `linkedin_list_mentionables` tool, with the same `q`/`limit` parameters and the same paid-plan requirement. See the [MCP Tools Reference](../mcp/tools-reference.md#linkedin_list_mentionables).

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad `x-publora-key` |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 403 | `"The mentionable-people directory is available on paid plans — upgrade to access it"` | Free plan — the response also includes `"code": "UPGRADE_REQUIRED"` |

> **Note:** API keys already require a paid plan, so API callers normally hit the `403` API-access error before the `UPGRADE_REQUIRED` one; the latter mainly appears in edge cases (e.g. a plan downgrade with a still-active key).

## Related

- [LinkedIn Mentions Guide](../guides/linkedin-mentions.md) — mention syntax, native vs web-profile ids, name matching
- [LinkedIn Comments](linkedin-comments.md) — mention people in comments
- [LinkedIn Reshare](linkedin-reshare.md) — mention people in reshare commentary
- [Create Post](create-post.md) — mention people in scheduled posts

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
