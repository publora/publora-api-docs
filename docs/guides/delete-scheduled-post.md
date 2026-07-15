# How to Delete a Scheduled Post Across All Platforms with Publora

Delete a scheduled post from Twitter, LinkedIn, Instagram, Threads, and any other platforms where it was scheduled—all in a single API call.

## Overview

When you create a post targeting multiple platforms, Publora creates a **post group** containing individual platform-specific posts. The delete endpoint removes the entire group atomically, ensuring consistent cleanup across all platforms.

## API Reference

**Endpoint:**
```
DELETE https://api.publora.com/api/v1/delete-post/:postGroupId
```

**Headers:**
```
x-publora-key: YOUR_API_KEY
```

**Path Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `postGroupId` | string | Yes | The post group ID returned when the post was created |

**Success Response (200):**
```json
{
  "success": true
}
```

## What Gets Deleted

A single delete request removes:

1. **The post group record** — Parent container for all platform posts
2. **All platform-specific posts** — Twitter, LinkedIn, Instagram, Threads, etc.
3. **All media file records** — Database references to images/videos
4. **All media files from storage** — Actual files in S3

```
Post Group (postGroupId: 507f1f77bcf86cd799439011)
├── Twitter post
├── LinkedIn post
└── Instagram post

DELETE /api/v1/delete-post/507f1f77bcf86cd799439011
→ All three platform posts deleted in one request
```

## Complete JavaScript Implementation

```javascript
const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY;
const BASE_URL = 'https://api.publora.com/api/v1';

/**
 * Custom error class for Publora API errors
 */
class PubloraError extends Error {
  constructor(message, statusCode, response) {
    super(message);
    this.name = 'PubloraError';
    this.statusCode = statusCode;
    this.response = response;
  }
}

/**
 * Delete a scheduled post across all platforms
 * @param {string} postGroupId - The post group ID to delete
 * @returns {Promise<{success: boolean}>} Deletion result
 * @throws {PubloraError} If deletion fails
 */
async function deleteScheduledPost(postGroupId) {
  // Validate input
  if (!postGroupId) {
    throw new Error('postGroupId is required');
  }

  if (typeof postGroupId !== 'string') {
    throw new Error('postGroupId must be a string');
  }

  // Make API request
  const response = await fetch(`${BASE_URL}/delete-post/${postGroupId}`, {
    method: 'DELETE',
    headers: {
      'x-publora-key': PUBLORA_API_KEY
    }
  });

  // Handle non-JSON responses (network errors, etc.)
  let data;
  try {
    data = await response.json();
  } catch (e) {
    throw new PubloraError(
      `Unexpected response: ${response.statusText}`,
      response.status,
      null
    );
  }

  // Handle API errors
  if (!response.ok) {
    let errorMessage = data.error || `HTTP ${response.status}`;

    switch (response.status) {
      case 401:
        errorMessage = 'Invalid API key. Check your x-publora-key header.';
        break;
      case 404:
        errorMessage = 'Post not found. It may have been deleted or belongs to another user.';
        break;
      case 500:
        errorMessage = 'Server error during deletion. The operation was rolled back.';
        break;
    }

    throw new PubloraError(errorMessage, response.status, data);
  }

  return data;
}

/**
 * Delete multiple posts with error tracking
 * @param {string[]} postGroupIds - Array of post group IDs to delete
 * @returns {Promise<{succeeded: string[], failed: Array<{postGroupId: string, error: string}>}>}
 */
async function deleteMultiplePosts(postGroupIds) {
  if (!Array.isArray(postGroupIds) || postGroupIds.length === 0) {
    throw new Error('postGroupIds must be a non-empty array');
  }

  const results = {
    succeeded: [],
    failed: []
  };

  for (const postGroupId of postGroupIds) {
    try {
      await deleteScheduledPost(postGroupId);
      results.succeeded.push(postGroupId);
    } catch (error) {
      results.failed.push({
        postGroupId,
        error: error.message,
        statusCode: error.statusCode
      });
    }
  }

  return results;
}

/**
 * Delete all scheduled posts for a specific platform
 * @param {string} platform - Platform name (twitter, linkedin, instagram, threads)
 * @returns {Promise<number>} Number of posts deleted
 */
async function deleteAllScheduledForPlatform(platform) {
  const validPlatforms = ['twitter', 'linkedin', 'instagram', 'threads', 'facebook'];
  if (!validPlatforms.includes(platform)) {
    throw new Error(`Invalid platform. Must be one of: ${validPlatforms.join(', ')}`);
  }

  let deletedCount = 0;
  let page = 1;

  while (true) {
    // Fetch scheduled posts for this platform
    const response = await fetch(
      `${BASE_URL}/list-posts?status=scheduled&platform=${platform}&page=${page}&limit=100`,
      { headers: { 'x-publora-key': PUBLORA_API_KEY } }
    );

    const data = await response.json();

    if (!data.posts || data.posts.length === 0) {
      break;
    }

    // Delete each post
    for (const post of data.posts) {
      try {
        await deleteScheduledPost(post.postGroupId);
        deletedCount++;
        console.log(`Deleted: ${post.postGroupId}`);
      } catch (error) {
        console.error(`Failed to delete ${post.postGroupId}: ${error.message}`);
      }
    }

    // Check for more pages
    if (!data.pagination?.hasNextPage) {
      break;
    }

    page++;
  }

  return deletedCount;
}

// ============ USAGE EXAMPLES ============

// Example 1: Delete a single post
async function deleteSinglePost() {
  try {
    const result = await deleteScheduledPost('507f1f77bcf86cd799439011');
    console.log('Post deleted successfully');
    return result;
  } catch (error) {
    if (error.statusCode === 404) {
      console.log('Post already deleted or not found');
    } else {
      console.error('Failed to delete:', error.message);
    }
    throw error;
  }
}

// Example 2: Delete multiple posts with summary
async function deleteBatch() {
  const postIds = [
    '507f1f77bcf86cd799439011',
    '507f1f77bcf86cd799439012',
    '507f1f77bcf86cd799439013'
  ];

  const results = await deleteMultiplePosts(postIds);

  console.log(`Succeeded: ${results.succeeded.length}`);
  console.log(`Failed: ${results.failed.length}`);

  if (results.failed.length > 0) {
    console.log('Failed deletions:');
    results.failed.forEach(f => console.log(`  ${f.postGroupId}: ${f.error}`));
  }

  return results;
}

// Example 3: Delete with confirmation (fetch details first)
async function deleteWithConfirmation(postGroupId) {
  // First, get post details
  const getResponse = await fetch(`${BASE_URL}/get-post/${postGroupId}`, {
    headers: { 'x-publora-key': PUBLORA_API_KEY }
  });

  if (!getResponse.ok) {
    throw new Error('Post not found');
  }

  const postData = await getResponse.json();
  const platforms = postData.posts.map(p => p.platform).join(', ');
  const content = postData.posts[0]?.content?.slice(0, 50) || '';

  console.log('About to delete:');
  console.log(`  Content: ${content}...`);
  console.log(`  Platforms: ${platforms}`);
  console.log(`  Status: ${postData.posts[0]?.status}`);

  // Perform deletion
  return await deleteScheduledPost(postGroupId);
}

// Run example
const result = await deleteSinglePost();
```

## Complete Python Implementation

```python
import os
import requests
from typing import List, Dict, Optional, Tuple
from dataclasses import dataclass

PUBLORA_API_KEY = os.environ.get('PUBLORA_API_KEY')
BASE_URL = 'https://api.publora.com/api/v1'


class PubloraError(Exception):
    """Custom exception for Publora API errors."""
    def __init__(self, message: str, status_code: int = None, response: dict = None):
        super().__init__(message)
        self.status_code = status_code
        self.response = response


@dataclass
class DeleteResult:
    """Result of a delete operation."""
    succeeded: List[str]
    failed: List[Dict[str, str]]


def delete_scheduled_post(post_group_id: str) -> Dict:
    """
    Delete a scheduled post across all platforms.

    Args:
        post_group_id: The post group ID to delete

    Returns:
        dict: Deletion result with success status

    Raises:
        PubloraError: If the API request fails
        ValueError: If inputs are invalid
    """
    # Validate input
    if not post_group_id:
        raise ValueError('post_group_id is required')

    if not isinstance(post_group_id, str):
        raise ValueError('post_group_id must be a string')

    # Make API request
    response = requests.delete(
        f'{BASE_URL}/delete-post/{post_group_id}',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    # Handle non-JSON responses
    try:
        data = response.json()
    except ValueError:
        raise PubloraError(
            f'Unexpected response: {response.text}',
            response.status_code,
            None
        )

    # Handle API errors
    if not response.ok:
        error_message = data.get('error', f'HTTP {response.status_code}')

        if response.status_code == 401:
            error_message = 'Invalid API key. Check your x-publora-key header.'
        elif response.status_code == 404:
            error_message = 'Post not found. It may have been deleted or belongs to another user.'
        elif response.status_code == 500:
            error_message = 'Server error during deletion. The operation was rolled back.'

        raise PubloraError(error_message, response.status_code, data)

    return data


def delete_multiple_posts(post_group_ids: List[str]) -> DeleteResult:
    """
    Delete multiple posts with error tracking.

    Args:
        post_group_ids: List of post group IDs to delete

    Returns:
        DeleteResult: Contains lists of succeeded and failed deletions
    """
    if not post_group_ids:
        raise ValueError('post_group_ids must be a non-empty list')

    succeeded = []
    failed = []

    for post_group_id in post_group_ids:
        try:
            delete_scheduled_post(post_group_id)
            succeeded.append(post_group_id)
        except (PubloraError, ValueError) as e:
            failed.append({
                'postGroupId': post_group_id,
                'error': str(e),
                'statusCode': getattr(e, 'status_code', None)
            })

    return DeleteResult(succeeded=succeeded, failed=failed)


def delete_all_scheduled_for_platform(platform: str) -> int:
    """
    Delete all scheduled posts that include a specific platform.

    Args:
        platform: Platform name (twitter, linkedin, instagram, threads)

    Returns:
        int: Number of posts deleted
    """
    valid_platforms = ['twitter', 'linkedin', 'instagram', 'threads', 'facebook']
    if platform not in valid_platforms:
        raise ValueError(f'Invalid platform. Must be one of: {", ".join(valid_platforms)}')

    deleted_count = 0
    page = 1

    while True:
        # Fetch scheduled posts for this platform
        response = requests.get(
            f'{BASE_URL}/list-posts',
            headers={'x-publora-key': PUBLORA_API_KEY},
            params={
                'status': 'scheduled',
                'platform': platform,
                'page': page,
                'limit': 100
            }
        )

        data = response.json()
        posts = data.get('posts', [])

        if not posts:
            break

        # Delete each post
        for post in posts:
            try:
                delete_scheduled_post(post['postGroupId'])
                deleted_count += 1
                print(f"Deleted: {post['postGroupId']}")
            except PubloraError as e:
                print(f"Failed to delete {post['postGroupId']}: {e}")

        # Check for more pages
        pagination = data.get('pagination', {})
        if not pagination.get('hasNextPage'):
            break

        page += 1

    return deleted_count


# ============ USAGE EXAMPLES ============

def delete_single_post_example():
    """Delete a single post."""
    try:
        result = delete_scheduled_post('507f1f77bcf86cd799439011')
        print('Post deleted successfully')
        return result
    except PubloraError as e:
        if e.status_code == 404:
            print('Post already deleted or not found')
        else:
            print(f'Failed to delete: {e}')
        raise


def delete_batch_example():
    """Delete multiple posts with summary."""
    post_ids = [
        '507f1f77bcf86cd799439011',
        '507f1f77bcf86cd799439012',
        '507f1f77bcf86cd799439013'
    ]

    results = delete_multiple_posts(post_ids)

    print(f'Succeeded: {len(results.succeeded)}')
    print(f'Failed: {len(results.failed)}')

    if results.failed:
        print('Failed deletions:')
        for f in results.failed:
            print(f"  {f['postGroupId']}: {f['error']}")

    return results


def delete_with_confirmation_example(post_group_id: str):
    """Delete with confirmation - fetch details first."""
    # First, get post details
    response = requests.get(
        f'{BASE_URL}/get-post/{post_group_id}',
        headers={'x-publora-key': PUBLORA_API_KEY}
    )

    if not response.ok:
        raise PubloraError('Post not found', response.status_code)

    post_data = response.json()
    platforms = ', '.join(p['platform'] for p in post_data.get('posts', []))
    content = post_data['posts'][0].get('content', '')[:50] if post_data.get('posts') else ''

    print('About to delete:')
    print(f'  Content: {content}...')
    print(f'  Platforms: {platforms}')
    print(f"  Status: {post_data['posts'][0].get('status') if post_data.get('posts') else 'unknown'}")

    # Perform deletion
    return delete_scheduled_post(post_group_id)


# Run examples
if __name__ == '__main__':
    # Delete single post
    result = delete_single_post_example()

    # Or delete all scheduled Twitter posts
    # count = delete_all_scheduled_for_platform('twitter')
    # print(f'Deleted {count} Twitter posts')
```

## cURL Examples

### Delete a single post

```bash
curl -X DELETE "https://api.publora.com/api/v1/delete-post/507f1f77bcf86cd799439011" \
  -H "x-publora-key: YOUR_API_KEY"

# Response:
# { "success": true }
```

### Bulk delete script

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

## Deletion Restrictions

| Post Status | Can Delete? | Notes |
|-------------|-------------|-------|
| `draft` | Yes | Not yet scheduled |
| `scheduled` | Yes | Before publication time |
| `published` | Yes | Deletes Publora's records only; it does not remove the live platform post |
| `failed` | Yes | Clean up failed attempts |
| `partially_published` | Yes | Deletes Publora's records only; already-published platform posts remain live |

`pending` and `processing` are values of the separate `processingStatus` field, not post-group statuses. Deleting a group while `processingStatus` is `pending` or `processing` removes Publora's records without cancelling in-flight platform work, which may still complete.

## Error Reference

| Status | Error | Cause | Solution |
|--------|-------|-------|----------|
| 401 | `"Invalid API key"` | Bad or missing `x-publora-key` | Check API key |
| 404 | `"Post group not found"` | ID doesn't exist or wrong user | Verify postGroupId |
| 500 | `"Failed to delete post group"` | Database transaction failed | Retry the request |

## Important Notes

### Atomic Deletion

The deletion uses a MongoDB transaction ensuring all database records (post group, platform posts, media records) are deleted atomically. If any database deletion fails, the entire operation is rolled back.

### Media Cleanup

Media files are deleted from S3 after the database transaction commits. S3 deletion failures are logged but do not fail the request—your post will still be successfully deleted.

### Rate Limiting

For bulk deletions, add a small delay between requests to avoid rate limits:

```javascript
for (const postId of postIds) {
  await deleteScheduledPost(postId);
  await new Promise(r => setTimeout(r, 100)); // 100ms delay
}
```

### Idempotent Behavior

Deleting an already-deleted post returns a 404 error. Check for this status code to handle idempotent deletion gracefully:

```javascript
try {
  await deleteScheduledPost(postGroupId);
} catch (error) {
  if (error.statusCode === 404) {
    // Already deleted, treat as success
    console.log('Post was already deleted');
  } else {
    throw error;
  }
}
```

## Related Guides

- [Create Post](../endpoints/create-post.md) - Create new posts
- [Get Post](../endpoints/get-post.md) - Check post status before deletion
- [Update Scheduled Post](update-scheduled-post.md) - Reschedule instead of delete
- [List Posts](../endpoints/list-posts.md) - Find posts to delete
