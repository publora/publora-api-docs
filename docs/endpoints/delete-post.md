# Delete Post

Delete an unpublished post group and its platform-specific records and media from Publora. Published content, uncertain publish outcomes, and publishing in progress are protected. This endpoint does **not** remove posts from social networks.

## Endpoint

```
DELETE https://api.publora.com/api/v1/delete-post/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Client identifier (e.g., `"mcp"`) |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID to delete |

## Response

Returns HTTP **200** with the following JSON body on success:

```json
{
  "success": true
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the deletion succeeded (only present on **200** responses) |

> **Note:** The `success` field only appears on successful (200) responses. Error responses return an `error` string field instead (e.g., `{ "error": "Post group not found" }`).

## What Gets Deleted

Before deleting anything, Publora checks ownership, the group state, and the state of every platform-specific post. A group waiting for a retry is protected if another platform has already published, may have published, or is still publishing.

For an eligible group, Publora reads the media references and conditionally removes the parent record only if its state has not changed. It then deletes the media records and platform-specific post records in the same database transaction. A concurrent scheduler claim or edit prevents the deletion.

After a successful commit, the associated files are removed from S3 storage.

### Deletion Behavior

- **Atomic operation**: The deletion uses a MongoDB transaction to ensure all database records (post group, platform posts, and media records) are deleted atomically. If any database deletion fails, the entire operation is rolled back.
- **S3 cleanup**: Media files are deleted from S3 storage after the database transaction completes. S3 deletion failures are logged but do not fail the request - your post will still be successfully deleted even if S3 cleanup encounters issues.

## Examples

### JavaScript - Delete Post Across All Platforms

```javascript
async function deletePostAcrossAllPlatforms(apiKey, postGroupId) {
  const response = await fetch(
    `https://api.publora.com/api/v1/delete-post/${postGroupId}`,
    {
      method: 'DELETE',
      headers: { 'x-publora-key': apiKey }
    }
  );

  if (!response.ok) {
    const error = await response.json();
    throw new Error(`Failed to delete: ${error.error || response.statusText}`);
  }

  const data = await response.json();
  console.log(`Successfully deleted post ${postGroupId}`);
  return data;
}

// Usage
await deletePostAcrossAllPlatforms('YOUR_API_KEY', '507f1f77bcf86cd799439011');
```

### JavaScript - Delete Multiple Posts

```javascript
async function deleteMultiplePosts(apiKey, postGroupIds) {
  const results = {
    succeeded: [],
    failed: []
  };

  for (const postGroupId of postGroupIds) {
    try {
      const response = await fetch(
        `https://api.publora.com/api/v1/delete-post/${postGroupId}`,
        {
          method: 'DELETE',
          headers: { 'x-publora-key': apiKey }
        }
      );

      if (response.ok) {
        results.succeeded.push({ postGroupId });
      } else {
        const error = await response.json();
        results.failed.push({ postGroupId, error: error.error });
      }
    } catch (err) {
      results.failed.push({ postGroupId, error: err.message });
    }
  }

  console.log(`Deleted ${results.succeeded.length} posts`);
  console.log(`Failed: ${results.failed.length}`);

  return results;
}

// Usage: Delete all posts from a list
const postIds = ['507f1f77bcf86cd799439011', '507f1f77bcf86cd799439012', '507f1f77bcf86cd799439013'];
await deleteMultiplePosts('YOUR_API_KEY', postIds);
```

### JavaScript - Delete with Confirmation

```javascript
async function deletePostWithConfirmation(apiKey, postGroupId) {
  // First, get post details to show what will be deleted
  const getResponse = await fetch(
    `https://api.publora.com/api/v1/get-post/${postGroupId}`,
    { headers: { 'x-publora-key': apiKey } }
  );

  if (!getResponse.ok) {
    throw new Error('Post not found');
  }

  const postData = await getResponse.json();

  console.log('About to delete:');
  console.log(`  Content: ${postData.posts[0]?.content?.slice(0, 50)}...`);
  console.log(`  Platforms: ${postData.posts.map(p => p.platform).join(', ')}`);
  console.log(`  Status: ${postData.posts[0]?.status}`);

  // Perform deletion
  const deleteResponse = await fetch(
    `https://api.publora.com/api/v1/delete-post/${postGroupId}`,
    {
      method: 'DELETE',
      headers: { 'x-publora-key': apiKey }
    }
  );

  const result = await deleteResponse.json();
  console.log('Deletion complete:', result);

  return result;
}
```

### Python - Delete Post Across All Platforms

```python
import requests
from typing import Dict, Any

def delete_post_across_all_platforms(api_key: str, post_group_id: str) -> Dict[str, Any]:
    """
    Delete a post from all platforms where it was scheduled.

    Args:
        api_key: Your Publora API key
        post_group_id: The post group ID to delete

    Returns:
        Deletion result with success status
    """
    response = requests.delete(
        f'https://api.publora.com/api/v1/delete-post/{post_group_id}',
        headers={'x-publora-key': api_key}
    )

    response.raise_for_status()
    data = response.json()

    print(f"Successfully deleted post {post_group_id}")
    return data


# Usage
result = delete_post_across_all_platforms('YOUR_API_KEY', '507f1f77bcf86cd799439011')
```

### Python - Bulk Delete with Error Handling

```python
import requests
from typing import List, Dict, Tuple

def bulk_delete_posts(api_key: str, post_group_ids: List[str]) -> Tuple[List[str], List[Dict]]:
    """
    Delete multiple posts, returning succeeded and failed results.

    Args:
        api_key: Your Publora API key
        post_group_ids: List of post group IDs to delete

    Returns:
        Tuple of (succeeded_ids, failed_list)
    """
    succeeded = []
    failed = []

    for post_group_id in post_group_ids:
        try:
            response = requests.delete(
                f'https://api.publora.com/api/v1/delete-post/{post_group_id}',
                headers={'x-publora-key': api_key}
            )

            if response.ok:
                succeeded.append(post_group_id)
            else:
                error = response.json()
                failed.append({
                    'postGroupId': post_group_id,
                    'error': error.get('error', 'Unknown error')
                })

        except Exception as e:
            failed.append({
                'postGroupId': post_group_id,
                'error': str(e)
            })

    print(f"Successfully deleted: {len(succeeded)}")
    print(f"Failed: {len(failed)}")

    return succeeded, failed


# Usage
post_ids = ['507f1f77bcf86cd799439011', '507f1f77bcf86cd799439012']
succeeded, failed = bulk_delete_posts('YOUR_API_KEY', post_ids)
```

### Python - Delete All Scheduled Posts for a Platform

```python
import requests

def delete_all_scheduled_for_platform(api_key: str, platform: str) -> int:
    """
    Delete all scheduled posts that include a specific platform.

    Args:
        api_key: Your Publora API key
        platform: Platform name (twitter, linkedin, instagram, etc.)

    Returns:
        Number of posts deleted
    """
    headers = {'x-publora-key': api_key}
    deleted_count = 0
    page = 1

    while True:
        # Fetch scheduled posts filtered by platform
        response = requests.get(
            'https://api.publora.com/api/v1/list-posts',
            headers=headers,
            params={
                'status': 'scheduled',
                'platform': platform,
                'page': page,
                'limit': 100
            }
        )

        data = response.json()

        if not data['posts']:
            break

        # Delete each post
        for post in data['posts']:
            delete_response = requests.delete(
                f"https://api.publora.com/api/v1/delete-post/{post['postGroupId']}",
                headers=headers
            )
            if delete_response.ok:
                deleted_count += 1
                print(f"Deleted {post['postGroupId']} from {platform}")

        if not data['pagination']['hasNextPage']:
            break

        page += 1

    print(f"Total posts deleted: {deleted_count}")
    return deleted_count


# Usage: Delete all scheduled Twitter posts
delete_all_scheduled_for_platform('YOUR_API_KEY', 'twitter')
```

### Node.js (axios) - Delete with Full Error Handling

```javascript
const axios = require('axios');

class PubloraClient {
  constructor(apiKey) {
    this.api = axios.create({
      baseURL: 'https://api.publora.com/api/v1',
      headers: { 'x-publora-key': apiKey }
    });
  }

  async deletePost(postGroupId) {
    try {
      await this.api.delete(`/delete-post/${postGroupId}`);
      return { success: true, postGroupId };
    } catch (error) {
      if (error.response) {
        return {
          success: false,
          postGroupId,
          error: error.response.data.error || 'Unknown error',
          status: error.response.status
        };
      }
      throw error;
    }
  }

  async deleteMultiplePosts(postGroupIds) {
    const results = await Promise.allSettled(
      postGroupIds.map(id => this.deletePost(id))
    );

    return results.map((result, index) => ({
      postGroupId: postGroupIds[index],
      ...(result.status === 'fulfilled' ? result.value : { success: false, error: result.reason.message })
    }));
  }
}

// Usage
const client = new PubloraClient('YOUR_API_KEY');

// Delete single post across all platforms
const result = await client.deletePost('507f1f77bcf86cd799439011');
console.log(`Deleted: ${result.success}`);

// Delete multiple posts
const results = await client.deleteMultiplePosts([
  '507f1f77bcf86cd799439011',
  '507f1f77bcf86cd799439012',
  '507f1f77bcf86cd799439013'
]);

const succeeded = results.filter(r => r.success);
const failed = results.filter(r => !r.success);
console.log(`Deleted: ${succeeded.length}, Failed: ${failed.length}`);
```

### cURL - Delete Post

```bash
# Delete a single post across all platforms
curl -X DELETE "https://api.publora.com/api/v1/delete-post/507f1f77bcf86cd799439011" \
  -H "x-publora-key: YOUR_API_KEY"

# Response:
# { "success": true }
```

### Bash - Bulk Delete Script

```bash
#!/bin/bash

API_KEY="YOUR_API_KEY"
POST_IDS=("507f1f77bcf86cd799439011" "507f1f77bcf86cd799439012" "507f1f77bcf86cd799439013")

SUCCEEDED=0
FAILED=0

for POST_ID in "${POST_IDS[@]}"; do
  RESPONSE=$(curl -s -w "\n%{http_code}" -X DELETE \
    "https://api.publora.com/api/v1/delete-post/${POST_ID}" \
    -H "x-publora-key: ${API_KEY}")

  HTTP_CODE=$(echo "$RESPONSE" | tail -n1)
  BODY=$(echo "$RESPONSE" | sed '$d')

  if [ "$HTTP_CODE" = "200" ]; then
    echo "Deleted $POST_ID successfully"
    SUCCEEDED=$((SUCCEEDED + 1))
  else
    echo "Failed to delete $POST_ID: $BODY"
    FAILED=$((FAILED + 1))
  fi
done

echo ""
echo "Summary: $SUCCEEDED succeeded, $FAILED failed"
```

## Errors

| Status | Response | Cause |
|--------|----------|-------|
| 400 | `{ "error": "Invalid post group ID" }` | The path parameter is not a valid MongoDB ObjectId |
| 400 | `{ "error": "Invalid x-publora-user-id" }` | The `x-publora-user-id` header value is not a valid ObjectId format |
| 401 | `{ "error": "API key is required" }` | Missing `x-publora-key` header |
| 401 | `{ "error": "Invalid API key" }` | `x-publora-key` value is incorrect or revoked |
| 401 | `{ "error": "Invalid API key owner" }` | The API key exists but its owner account could not be found |
| 403 | `{ "error": "API access is not enabled for this account" }` | No active subscription or API access not enabled |
| 403 | `{ "error": "MCP access is not enabled for this account" }` | The account does not have MCP access enabled (MCP-only keys) |
| 403 | `{ "error": "Workspace access is not enabled for this key" }` | The API key does not have workspace/managed-user permissions |
| 403 | `{ "error": "User is not managed by key" }` | The `x-publora-user-id` references a user not managed by this API key |
| 404 | `{ "error": "Post group not found" }` | The post group ID doesn't exist or belongs to another user |
| 409 | `{ "error": "...", "code": "POST_IS_PUBLISHED" }` | The group is fully published; API deletion is not allowed |
| 409 | `{ "error": "...", "code": "POST_HAS_LIVE_CONTENT" }` | Some content is live or its publish outcome is unknown, including while another platform waits for a retry |
| 409 | `{ "error": "...", "code": "POST_IS_PROCESSING" }` | The group or one of its platform-specific posts is being published |
| 409 | `{ "error": "...", "code": "POST_CHANGED" }` | A concurrent edit changed the group before deletion; fetch the current state before trying again |
| 500 | `{ "error": "Failed to delete post group" }` | Server error during database transaction (deletion is rolled back) |
| 500 | `{ "error": "Internal server error" }` | Unexpected server error in middleware |

> **Note:** If `x-publora-user-id` matches the API key owner, no workspace check is triggered — the header is effectively a no-op in that case.

## Important Notes

### Cross-Platform Deletion

When you create a post with multiple platforms (e.g., Twitter, LinkedIn, and Instagram), Publora creates a **post group** containing individual platform-specific posts. Deleting via `postGroupId` removes the entire group - you don't need to delete each platform separately.

```
Post Group (postGroupId: 507f1f77bcf86cd799439011)
├── Twitter post
├── LinkedIn post
└── Instagram post

DELETE /delete-post/507f1f77bcf86cd799439011
→ Deletes all three Publora records if none is published or being published
```

### Malformed Post Group IDs

If the `postGroupId` is not a valid MongoDB ObjectId format (e.g., too short, contains invalid characters), the server returns **400** with `{ "error": "Invalid post group ID" }`. Ensure you pass only valid ObjectId strings received from the create-post or list-posts endpoints.

### Protected Publication States

Draft, scheduled, and failed groups can be deleted only when their platform-specific posts have no evidence of publication or active publishing. A `scheduled` group is not necessarily unpublished: one platform may have succeeded while another waits for a retry.

Fully published and partially published groups return **409**. A failed thread with published parts, a stored post ID or published timestamp, and an unknown publish outcome are also protected. The group, platform-specific records, and media remain intact when deletion is blocked.

This changes the previous API behavior that allowed published-history cleanup. There is no force-delete option. Handle the structured `code` field rather than matching the human-readable error text:

- `POST_IS_PUBLISHED` / `POST_HAS_LIVE_CONTENT`: keep the published history. To recover or edit the content, create a separate draft. Deleting a Publora record cannot unpublish a social-network post.
- `POST_IS_PROCESSING`: wait for publishing to finish, then fetch the current state. A completed publication remains protected.
- `POST_CHANGED`: fetch the current state before deciding whether to retry the deletion.

Example blocked response:

```json
{
  "error": "Published posts cannot be deleted through this operation.",
  "code": "POST_IS_PUBLISHED"
}
```

### Media File Cleanup

All media files (images, videos) associated with the deleted post are automatically removed from both the database and S3 storage. The database deletion happens atomically within a MongoDB transaction. S3 file deletion occurs after the transaction commits - if S3 deletion fails, it is logged but does not affect the success of your delete request.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
