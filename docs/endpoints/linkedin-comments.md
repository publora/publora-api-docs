# LinkedIn Comments

Create and delete comments on LinkedIn posts programmatically.

## Create Comment

```
POST https://api.publora.com/api/v1/linkedin-comments
```

### Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `Content-Type` | Yes | `application/json` |

### Request Body

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN: `urn:li:share:xxx`, `urn:li:ugcPost:xxx`, or `urn:li:activity:xxx` |
| `message` | string | Yes | Raw input up to 10,000 characters. After mention syntax is converted, the text sent to LinkedIn must be at most 1,250 characters. |
| `platformId` | string | Yes | LinkedIn connection ID (format: `linkedin-ABC123`) |
| `parentComment` | string | No | Parent comment URN for threaded replies |

### Response (HTTP 201 Created)

```json
{
  "success": true,
  "comment": {
    "id": "6636062862760562688",
    "commentUrn": "urn:li:comment:(urn:li:activity:7123456789012345678,6636062862760562688)",
    "message": "Great post! Thanks for sharing.",
    "created": {
      "time": 1710345600000
    }
  }
}
```

> **Note:** The `id` field is the numeric comment ID extracted from the LinkedIn API's `x-restli-id` response header. The full comment URN is in the `commentUrn` field. The `comment` object also includes data returned by the LinkedIn API spread into it, so additional fields from the LinkedIn response may be present.

> **Note:** Fields like `created` are passed through from LinkedIn's API response and may vary. They are not guaranteed to always be present.

### Examples

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    message: 'Great post! Thanks for sharing.',
    platformId: 'linkedin-Tz9W5i6ZYG'
  })
});
const data = await response.json();
console.log(data.comment.id);
```

#### Python (requests)

```python
import requests

response = requests.post(
    'https://api.publora.com/api/v1/linkedin-comments',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7123456789012345678',
        'message': 'Great post! Thanks for sharing.',
        'platformId': 'linkedin-Tz9W5i6ZYG'
    }
)
print(response.json()['comment']['id'])
```

#### cURL

```bash
curl -X POST https://api.publora.com/api/v1/linkedin-comments \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7123456789012345678",
    "message": "Great post! Thanks for sharing.",
    "platformId": "linkedin-Tz9W5i6ZYG"
  }'
```

### Reply to a Comment

To create a threaded reply, include the `parentComment` parameter:

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    message: 'Thanks for the kind words!',
    platformId: 'linkedin-Tz9W5i6ZYG',
    parentComment: 'urn:li:comment:(urn:li:activity:7123456789012345678,6636062862760562688)'
  })
});
```

---

## Delete Comment

```
DELETE https://api.publora.com/api/v1/linkedin-comments
```

### Request Body or Query Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postedId` | string | Yes | LinkedIn post URN |
| `commentId` | string | Yes | Comment ID or URN to delete |
| `platformId` | string | Yes | LinkedIn connection ID |

### Response

```json
{
  "success": true,
  "deleted": "6636062862760562688"
}
```

The `deleted` field contains the comment ID that was removed. This can be a numeric ID or a full URN (e.g., `urn:li:comment:(urn:li:activity:123,6636062862760562688)`), depending on what was passed as `commentId`.

### Examples

#### JavaScript (fetch)

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'DELETE',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    commentId: '6636062862760562688',
    platformId: 'linkedin-Tz9W5i6ZYG'
  })
});
```

#### Python (requests)

```python
response = requests.delete(
    'https://api.publora.com/api/v1/linkedin-comments',
    headers={
        'Content-Type': 'application/json',
        'x-publora-key': 'YOUR_API_KEY'
    },
    json={
        'postedId': 'urn:li:share:7123456789012345678',
        'commentId': '6636062862760562688',
        'platformId': 'linkedin-Tz9W5i6ZYG'
    }
)
```

#### cURL

```bash
curl -X DELETE "https://api.publora.com/api/v1/linkedin-comments" \
  -H "Content-Type: application/json" \
  -H "x-publora-key: YOUR_API_KEY" \
  -d '{
    "postedId": "urn:li:share:7123456789012345678",
    "commentId": "6636062862760562688",
    "platformId": "linkedin-Tz9W5i6ZYG"
  }'
```

## Mentions in Comments

You can @mention LinkedIn members and organizations in comments using the same syntax as posts:

```
@{urn:li:person:MEMBER_ID|Display Name}       # Mention a person
@{urn:li:organization:ORG_ID|Company Name}    # Mention an organization
```

Publora automatically converts this to LinkedIn's comment mention format (`message.attributes`). The mention renders as a clickable link and notifies the mentioned user.

### Example with Mention

```javascript
const response = await fetch('https://api.publora.com/api/v1/linkedin-comments', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': 'YOUR_API_KEY'
  },
  body: JSON.stringify({
    postedId: 'urn:li:share:7123456789012345678',
    message: 'Great point @{urn:li:person:ACoAABcD1234EfG|Jane Smith}! Totally agree.',
    platformId: 'linkedin-Tz9W5i6ZYG'
  })
});
```

**Result on LinkedIn:** `Great point @Jane Smith! Totally agree.` — with "Jane Smith" as a clickable profile link.

> **Important:** You must use a valid LinkedIn URN ID. Invalid IDs will cause LinkedIn to reject the comment with a `400` error. See the [LinkedIn Mentions Guide](../guides/linkedin-mentions.md) for how to find URN IDs.

### postedId Format

Publora accepts `urn:li:share:xxx`, `urn:li:ugcPost:xxx`, and `urn:li:activity:xxx`. It first sends the supplied URN to LinkedIn. If LinkedIn rejects an activity URN with a retryable target error, Publora also attempts the corresponding `ugcPost` and `share` URNs with the same ID. LinkedIn can still reject a URN based on the target post or account permissions.

For posts created via Publora, prefer the exact `postedId` returned by [get-post](get-post.md).

---

## Errors

### Create Comment Errors (POST)

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId, message, and platformId are required"` | Missing required fields |
| 400 | `"platformId must be a string"` | platformId is not a string type |
| 400 | `"postedId must be a valid LinkedIn URN"` | Not a valid LinkedIn URN format |
| 400 | `"message cannot be empty"` | Message is an empty string |
| 400 | `"comment text cannot exceed 10000 characters"` | Raw input exceeds the defensive pre-processing cap |
| 400 | `"comment text cannot exceed 1250 characters after mention processing"` | Converted LinkedIn comment text exceeds the platform limit |
| 400 | `"parentComment must be a valid LinkedIn URN"` | parentComment is not a valid URN format |
| 400 | `"Invalid platformId"` | platformId format is invalid |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to create LinkedIn comment"` | Server error while creating comment |

### Delete Comment Errors (DELETE)

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"postedId, commentId, and platformId are required"` | Missing required fields |
| 400 | `"platformId must be a string"` | platformId is not a string type |
| 400 | `"postedId must be a valid LinkedIn URN"` | Not a valid LinkedIn URN format |
| 400 | `"commentId must be a valid LinkedIn URN or numeric ID"` | commentId format is invalid |
| 400 | `"Invalid platformId"` | platformId format is invalid |
| 401 | `"API key is required"` | No `x-publora-key` header provided |
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` |
| 401 | `"Invalid API key owner"` | API key owner user not found in database |
| 403 | `"API access is not enabled for this account"` | Account does not have API access enabled |
| 404 | `"LinkedIn connection not found"` | No LinkedIn account with that platformId |
| 500 | `"Failed to delete LinkedIn comment"` | Server error while deleting comment |

> **Note:** The 500 error response includes a `details` field with the underlying error message: `{ "error": "Failed to delete LinkedIn comment", "details": "<error message>" }`.

> **Note:** Error status codes from the LinkedIn API may be forwarded directly (e.g., 403, 429), so you may receive error codes other than those listed above.

## Implementation Notes

- **Create comment response field precedence:** The create comment response constructs the `comment` object by spreading `...response.data` from the LinkedIn API, then explicitly setting `id`, `commentUrn`, and `message`. If LinkedIn ever returns fields named `id`, `commentUrn`, or `message` in its response body, the explicit values will override them. However, if LinkedIn adds new fields with those names in a different format, the spread could briefly surface unexpected data before the override. In practice this is unlikely but worth noting for defensive parsing.

- **Delete comment `commentId` format:** The REST API accepts both a full URN (e.g., `urn:li:comment:(urn:li:activity:123,456)`) and a plain numeric comment ID (e.g., `"6636062862760562688"`). The MCP tool description only mentions URN format, but both formats work identically via the REST API.

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) -- the best AI service for authentic thought leadership at scale.*
