# TypeScript Quick Start

Type-safe integration with the Publora API using TypeScript and modern tooling.

## Installation

```bash
npm install typescript ts-node @types/node
```

## Type Definitions

```typescript
// types/publora.ts

export interface PlatformConnection {
  platformId: string;
  platform: 'twitter' | 'linkedin' | 'instagram' | 'threads' | 'tiktok' | 'youtube' | 'facebook' | 'bluesky' | 'mastodon' | 'telegram';
  username: string;
  displayName: string;
  profileImageUrl?: string;
  accessTokenExpiresAt?: string;
}

export interface Post {
  _id: string;
  platform: string;
  platformId: string;
  content: string;
  status: 'draft' | 'scheduled' | 'pending' | 'processing' | 'published' | 'failed';
  postedId?: string;
  publishedUrl?: string;
  error?: string;
}

export interface PostGroup {
  postGroupId: string;
  content: string;
  status: 'draft' | 'scheduled' | 'pending' | 'processing' | 'published' | 'partially_published' | 'failed';
  scheduledTime?: string;
  posts: Post[];
}

export interface CreatePostRequest {
  content: string;
  platforms: string[];
  scheduledTime?: string;
  status?: 'draft' | 'scheduled';
  platformSettings?: PlatformSettings;
}

export interface PlatformSettings {
  instagram?: {
    videoType?: 'REELS' | 'STORIES';
  };
  tiktok?: {
    disableDuet?: boolean;
    disableStitch?: boolean;
    disableComment?: boolean;
  };
  telegram?: {
    parseMode?: 'HTML' | 'MarkdownV2';
    disableWebPagePreview?: boolean;
  };
}

export interface CreatePostResponse {
  success: boolean;
  postGroupId: string;
}

export interface ApiError {
  error?: string;
  message?: string;
}

export interface UploadUrlResponse {
  uploadUrl: string;
  fileUrl: string;
  mediaId: string;
}

export interface LinkedInStats {
  success: boolean;
  metrics: {
    IMPRESSION: number;
    MEMBERS_REACHED: number;
    RESHARE: number;
    REACTION: number;
    COMMENT: number;
  };
  cached: boolean;
}
```

## API Client Class

```typescript
// lib/publora-client.ts

import type {
  PlatformConnection,
  PostGroup,
  CreatePostRequest,
  CreatePostResponse,
  UploadUrlResponse,
  LinkedInStats,
  ApiError
} from '../types/publora';

export class PubloraApiError extends Error {
  constructor(
    public status: number,
    message: string,
    public body?: ApiError
  ) {
    super(message);
    this.name = 'PubloraApiError';
  }
}

export class PubloraClient {
  private readonly baseUrl = 'https://api.publora.com/api/v1';
  private readonly headers: HeadersInit;

  constructor(
    private readonly apiKey: string,
    private readonly userId?: string
  ) {
    this.headers = {
      'Content-Type': 'application/json',
      'x-publora-key': apiKey,
      ...(userId && { 'x-publora-user-id': userId })
    };
  }

  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const response = await fetch(`${this.baseUrl}${endpoint}`, {
      ...options,
      headers: { ...this.headers, ...options.headers }
    });

    const body = await response.json();

    if (!response.ok) {
      const message = body.error || body.message || 'Unknown API error';
      throw new PubloraApiError(response.status, message, body);
    }

    return body as T;
  }

  async getConnections(): Promise<PlatformConnection[]> {
    const data = await this.request<{ connections: PlatformConnection[] }>(
      '/platform-connections'
    );
    return data.connections;
  }

  async createPost(request: CreatePostRequest): Promise<CreatePostResponse> {
    return this.request<CreatePostResponse>('/create-post', {
      method: 'POST',
      body: JSON.stringify(request)
    });
  }

  async getPost(postGroupId: string): Promise<PostGroup> {
    return this.request<PostGroup>(`/get-post/${postGroupId}`);
  }

  async updatePost(
    postGroupId: string,
    updates: Partial<CreatePostRequest>
  ): Promise<PostGroup> {
    return this.request<PostGroup>(`/update-post/${postGroupId}`, {
      method: 'PUT',
      body: JSON.stringify(updates)
    });
  }

  async deletePost(postGroupId: string): Promise<{ success: boolean }> {
    return this.request<{ success: boolean }>(`/delete-post/${postGroupId}`, {
      method: 'DELETE'
    });
  }

  async getUploadUrl(
    fileName: string,
    contentType: string,
    postGroupId: string
  ): Promise<UploadUrlResponse> {
    return this.request<UploadUrlResponse>('/get-upload-url', {
      method: 'POST',
      body: JSON.stringify({ fileName, contentType, postGroupId })
    });
  }

  async getLinkedInPostStats(
    platformId: string,
    postedId: string
  ): Promise<LinkedInStats> {
    return this.request<LinkedInStats>('/linkedin-post-statistics', {
      method: 'POST',
      body: JSON.stringify({ platformId, postedId, queryTypes: 'ALL' })
    });
  }
}
```

## Usage Examples

### Initialize Client

```typescript
import { PubloraClient } from './lib/publora-client';

const client = new PubloraClient(process.env.PUBLORA_API_KEY!);
```

### List Platform Connections

```typescript
async function listConnections(): Promise<void> {
  const connections = await client.getConnections();

  console.log(`Found ${connections.length} connected accounts:`);

  for (const conn of connections) {
    console.log(`  - ${conn.platform}: ${conn.username} (${conn.platformId})`);
  }
}

listConnections();
```

### Create a Post with Type Safety

```typescript
import type { CreatePostRequest } from './types/publora';

async function createTypeSafePost(): Promise<void> {
  const request: CreatePostRequest = {
    content: 'Hello from TypeScript! Type-safe social media posting.',
    platforms: ['twitter-123456789', 'linkedin-ABC123DEF'],
    scheduledTime: new Date(Date.now() + 3600000).toISOString() // 1 hour from now
  };

  const response = await client.createPost(request);

  console.log('Post created:', response.postGroupId);
}

createTypeSafePost();
```

### Schedule Multiple Posts

```typescript
interface ScheduledPost {
  content: string;
  platforms: string[];
  scheduledTime: Date;
}

async function scheduleWeek(posts: ScheduledPost[]): Promise<string[]> {
  const postGroupIds: string[] = [];

  for (const post of posts) {
    const response = await client.createPost({
      content: post.content,
      platforms: post.platforms,
      scheduledTime: post.scheduledTime.toISOString()
    });

    postGroupIds.push(response.postGroupId);
    console.log(`Scheduled: ${post.content.slice(0, 30)}... -> ${response.postGroupId}`);

    // Rate limiting
    await new Promise(resolve => setTimeout(resolve, 200));
  }

  return postGroupIds;
}

// Example usage
const weeklyContent: ScheduledPost[] = [
  {
    content: 'Monday motivation: Start your week with purpose!',
    platforms: ['twitter-123456789'],
    scheduledTime: new Date('2026-03-01T09:00:00Z')
  },
  {
    content: 'Tech tip Tuesday: Always version your APIs.',
    platforms: ['twitter-123456789', 'linkedin-ABC123DEF'],
    scheduledTime: new Date('2026-03-02T09:00:00Z')
  },
  {
    content: 'Wednesday wisdom: Ship fast, iterate faster.',
    platforms: ['twitter-123456789'],
    scheduledTime: new Date('2026-03-03T09:00:00Z')
  }
];

scheduleWeek(weeklyContent);
```

### Post with Platform-Specific Settings

```typescript
async function postInstagramReel(): Promise<void> {
  const response = await client.createPost({
    content: 'Behind the scenes! #buildinpublic',
    platforms: ['instagram-789012345'],
    platformSettings: {
      instagram: {
        videoType: 'REELS'
      }
    }
  });

  console.log('Instagram Reel scheduled:', response.postGroupId);
}

async function postToTelegram(): Promise<void> {
  const response = await client.createPost({
    content: '*Bold* and _italic_ text with [link](https://example.com)',
    platforms: ['telegram-1001234567890'],
    platformSettings: {
      telegram: {
        parseMode: 'MarkdownV2',
        disableWebPagePreview: false
      }
    }
  });

  console.log('Telegram message scheduled:', response.postGroupId);
}
```

### Upload Media with Type Safety

```typescript
import * as fs from 'fs';
import * as path from 'path';

async function uploadAndPost(filePath: string, caption: string): Promise<void> {
  const fileName = path.basename(filePath);
  const contentType = getMimeType(fileName);

  // Step 1: Create the post first
  const postResponse = await client.createPost({
    content: caption,
    platforms: ['twitter-123456789', 'linkedin-ABC123DEF']
  });

  // Step 2: Get upload URL with postGroupId
  const { uploadUrl, fileUrl, mediaId } = await client.getUploadUrl(fileName, contentType, postResponse.postGroupId);

  // Step 3: Upload to S3
  const fileBuffer = fs.readFileSync(filePath);
  await fetch(uploadUrl, {
    method: 'PUT',
    headers: { 'Content-Type': contentType },
    body: fileBuffer
  });

  console.log('Post with media created:', postResponse.postGroupId);
  console.log('File URL:', fileUrl);
}

function getMimeType(fileName: string): string {
  const ext = path.extname(fileName).toLowerCase();
  const mimeTypes: Record<string, string> = {
    '.jpg': 'image/jpeg',
    '.jpeg': 'image/jpeg',
    '.png': 'image/png',
    '.gif': 'image/gif',
    '.mp4': 'video/mp4',
    '.webp': 'image/webp'
  };
  return mimeTypes[ext] || 'application/octet-stream';
}

uploadAndPost('./screenshot.png', 'Check out our latest feature!');
```

### Error Handling with Types

```typescript
import { PubloraApiError } from './lib/publora-client';

async function safeCreatePost(): Promise<void> {
  try {
    const response = await client.createPost({
      content: 'Test post',
      platforms: ['twitter-123456789']
    });

    console.log('Success:', response.postGroupId);

  } catch (error) {
    if (error instanceof PubloraApiError) {
      switch (error.status) {
        case 401:
          console.error('Authentication failed. Check your API key.');
          break;
        case 403:
          console.error('Access denied:', error.message);
          // Handle subscription or limit issues
          break;
        case 400:
          console.error('Bad request:', error.message);
          // Handle validation errors
          break;
        case 404:
          console.error('Not found:', error.message);
          break;
        case 429:
          console.error('Rate limited. Retry after delay.');
          break;
        case 500:
          console.error('Server error. Retry later.');
          break;
        default:
          console.error(`API error (${error.status}):`, error.message);
      }
    } else if (error instanceof Error) {
      console.error('Network or unexpected error:', error.message);
    }
  }
}
```

### Monitor Post Status

```typescript
async function waitForPublish(
  postGroupId: string,
  maxAttempts: number = 30,
  intervalMs: number = 10000
): Promise<PostGroup> {
  for (let attempt = 0; attempt < maxAttempts; attempt++) {
    const postGroup = await client.getPost(postGroupId);

    console.log(`[${attempt + 1}/${maxAttempts}] Status: ${postGroup.status}`);

    if (postGroup.status === 'published') {
      console.log('All platforms published successfully!');
      return postGroup;
    }

    if (postGroup.status === 'partially_published') {
      const failed = postGroup.posts.filter(p => p.status === 'failed');
      const published = postGroup.posts.filter(p => p.status === 'published');

      console.log(`Published: ${published.length}, Failed: ${failed.length}`);
      failed.forEach(p => console.log(`  - ${p.platform}: ${p.error}`));

      return postGroup;
    }

    if (postGroup.status === 'failed') {
      console.log('All platforms failed:');
      postGroup.posts.forEach(p => console.log(`  - ${p.platform}: ${p.error}`));
      return postGroup;
    }

    await new Promise(resolve => setTimeout(resolve, intervalMs));
  }

  throw new Error(`Timeout waiting for post ${postGroupId} to publish`);
}

// Usage
const response = await client.createPost({
  content: 'Publishing now!',
  platforms: ['twitter-123456789']
});

const finalStatus = await waitForPublish(response.postGroupId);
console.log('Final status:', finalStatus.status);
```

### Batch Operations with Generics

```typescript
async function batchOperation<T, R>(
  items: T[],
  operation: (item: T) => Promise<R>,
  delayMs: number = 200
): Promise<Array<{ item: T; result: R | null; error: string | null }>> {
  const results: Array<{ item: T; result: R | null; error: string | null }> = [];

  for (const item of items) {
    try {
      const result = await operation(item);
      results.push({ item, result, error: null });
    } catch (err) {
      const error = err instanceof Error ? err.message : 'Unknown error';
      results.push({ item, result: null, error });
    }

    await new Promise(resolve => setTimeout(resolve, delayMs));
  }

  return results;
}

// Example: Batch delete posts
async function batchDeletePosts(postGroupIds: string[]): Promise<void> {
  const results = await batchOperation(
    postGroupIds,
    (id) => client.deletePost(id)
  );

  const succeeded = results.filter(r => r.result !== null).length;
  const failed = results.filter(r => r.error !== null);

  console.log(`Deleted ${succeeded}/${postGroupIds.length} posts`);

  if (failed.length > 0) {
    console.log('Failed:');
    failed.forEach(f => console.log(`  - ${f.item}: ${f.error}`));
  }
}
```

### LinkedIn Analytics

```typescript
async function generateLinkedInReport(
  platformId: string,
  postedIds: string[]
): Promise<void> {
  console.log('=== LinkedIn Analytics Report ===\n');

  let totalImpressions = 0;
  let totalEngagement = 0;

  for (const postedId of postedIds) {
    try {
      const stats = await client.getLinkedInPostStats(platformId, postedId);

      const engagement = stats.metrics.REACTION + stats.metrics.COMMENT + stats.metrics.RESHARE;
      totalImpressions += stats.metrics.IMPRESSION;
      totalEngagement += engagement;

      console.log(`Post: ${postedId}`);
      console.log(`  Impressions: ${stats.metrics.IMPRESSION.toLocaleString()}`);
      console.log(`  Engagement: ${engagement} (${stats.metrics.REACTION} reactions, ${stats.metrics.COMMENT} comments, ${stats.metrics.RESHARE} shares)`);
      console.log('');

    } catch (error) {
      console.log(`Post: ${postedId} - Error: ${error instanceof Error ? error.message : 'Unknown'}`);
    }
  }

  console.log('=== TOTALS ===');
  console.log(`Total Impressions: ${totalImpressions.toLocaleString()}`);
  console.log(`Total Engagement: ${totalEngagement}`);
  console.log(`Avg Engagement Rate: ${((totalEngagement / totalImpressions) * 100).toFixed(2)}%`);
}
```

## Best Practices

1. **Use strict TypeScript** - Enable `strict` mode in `tsconfig.json` for maximum type safety
2. **Define all types** - Create interfaces for all API requests and responses
3. **Handle all error types** - Use type guards to properly handle `PubloraApiError` vs network errors
4. **Use environment variables** - Never hardcode API keys; use `process.env.PUBLORA_API_KEY`
5. **Add JSDoc comments** - Document functions for better IDE support

```typescript
/**
 * Creates a scheduled post across multiple platforms.
 * @param content - The post content (max 280 chars for Twitter)
 * @param platforms - Array of platform IDs to post to
 * @param scheduledTime - ISO 8601 timestamp for scheduling (must be in future)
 * @returns Promise resolving to the created post group ID
 * @throws PubloraApiError if the request fails
 */
async function createScheduledPost(
  content: string,
  platforms: string[],
  scheduledTime: Date
): Promise<string> {
  const response = await client.createPost({
    content,
    platforms,
    scheduledTime: scheduledTime.toISOString()
  });
  return response.postGroupId;
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
