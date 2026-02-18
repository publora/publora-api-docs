# Delete Post

Delete a scheduled or draft post across all platforms where it was scheduled. This endpoint removes the entire post group and all associated platform-specific posts in a single operation.

## Endpoint

```
DELETE https://api.publora.com/api/v1/delete-post/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | The post group ID to delete |

## Response

```json
{
  "success": true
}
```

## Response Fields

| Field | Type | Description |
|-------|------|-------------|
| `success` | boolean | Whether the deletion succeeded |

## What Gets Deleted

When you delete a post group, the following are removed:

1. **The post group record** - The parent container for all platform posts
2. **All platform-specific posts** - Twitter post, LinkedIn post, Instagram post, etc.
3. **All media file records from database** - References to media in storage
4. **All media files from S3 storage** - The actual image and video files are deleted from S3

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
| 401 | `{ "error": "Invalid API key" }` | Bad or missing `x-publora-key` |
| 404 | `{ "error": "Post group not found" }` | The post group ID doesn't exist or belongs to another user |
| 500 | `{ "error": "Failed to delete post group" }` | Server error during database transaction (deletion is rolled back) |

## Important Notes

### Cross-Platform Deletion

When you create a post with multiple platforms (e.g., Twitter, LinkedIn, and Instagram), Publora creates a **post group** containing individual platform-specific posts. Deleting via `postGroupId` removes the entire group - you don't need to delete each platform separately.

```
Post Group (postGroupId: 507f1f77bcf86cd799439011)
├── Twitter post
├── LinkedIn post
└── Instagram post

DELETE /delete-post/507f1f77bcf86cd799439011
→ Deletes ALL THREE platform posts in one request
```

### Deletion Restrictions

- **Drafts** can be deleted at any time
- **Scheduled posts** can be deleted before their scheduled time
- **Published posts** cannot be deleted via API (they already exist on the platforms)
- **Failed posts** can be deleted to clean up your post list

### Media File Cleanup

All media files (images, videos) associated with the deleted post are automatically removed from both the database and S3 storage. The database deletion happens atomically within a MongoDB transaction. S3 file deletion occurs after the transaction commits - if S3 deletion fails, it is logged but does not affect the success of your delete request.


---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
