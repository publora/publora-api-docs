# Using Publora with Cursor AI

Get the most out of AI-assisted development when building with the Publora API.

## Overview

Cursor is an AI-powered code editor that can help you write, understand, and debug code faster. This guide shows you how to effectively use Cursor with Publora's API.

## Setting Up Context

### Add Publora to Your Project Rules

Create a `.cursorrules` file in your project root to give Cursor context about Publora:

```text
# .cursorrules

## Publora API Integration

This project uses Publora API for social media scheduling.

### API Details
- Base URL: https://api.publora.com/api/v1
- Auth: x-publora-key header with API key
- Docs: https://docs.publora.com

### Key Endpoints
- GET /platform-connections - List connected accounts
- POST /create-post - Create/schedule posts
- GET /get-post/{id} - Get post status
- PUT /update-post/{id} - Update scheduled post
- DELETE /delete-post/{id} - Delete post
- POST /get-upload-url - Get pre-signed upload URL
- POST /linkedin-post-statistics - Get LinkedIn analytics

### Platform IDs
Format: {platform}-{id} (e.g., twitter-123456789, linkedin-ABC123DEF)

### Supported Platforms
twitter, linkedin, instagram, threads, tiktok, youtube, facebook, bluesky, mastodon, telegram

### Request Format
- Content-Type: application/json
- scheduledTime: ISO 8601 UTC format (YYYY-MM-DDTHH:mm:ss.sssZ)
- platforms: array of platform IDs

### Common Patterns
- Always check response.ok or status code before processing
- Use 200ms delay between batch requests for rate limiting
- Handle partial failures (some platforms may fail while others succeed)
```

### Example Prompts for Cursor

When working with Cursor, use these prompts to get accurate Publora code:

```text
# Good prompts for Cursor

"Create a function to schedule a post to Twitter and LinkedIn using Publora API"

"Add error handling for Publora API responses with retry logic for 500 errors"

"Build a React component that shows connected social accounts from Publora"

"Write a Python script to import posts from CSV and schedule them via Publora"

"Create a Next.js API route to proxy Publora requests with proper authentication"
```

## Code Snippets for Cursor

### Quick Reference: JavaScript/TypeScript

```typescript
// Cursor: Use this as reference for Publora API calls

// Authentication header
const headers = {
  'Content-Type': 'application/json',
  'x-publora-key': process.env.PUBLORA_API_KEY,
};

// List connections
const connections = await fetch('https://api.publora.com/api/v1/platform-connections', {
  headers
}).then(r => r.json());

// Create post
const post = await fetch('https://api.publora.com/api/v1/create-post', {
  method: 'POST',
  headers,
  body: JSON.stringify({
    content: 'Your post content',
    platforms: ['twitter-123456789'],
    scheduledTime: '2026-03-01T14:00:00.000Z', // Optional
    mediaUrls: ['https://example.com/image.jpg'], // Optional
  })
}).then(r => r.json());

// Get post status
const status = await fetch(`https://api.publora.com/api/v1/get-post/${postGroupId}`, {
  headers
}).then(r => r.json());

// Delete post
await fetch(`https://api.publora.com/api/v1/delete-post/${postGroupId}`, {
  method: 'DELETE',
  headers
});
```

### Quick Reference: Python

```python
# Cursor: Use this as reference for Publora API calls

import requests

BASE_URL = 'https://api.publora.com/api/v1'
headers = {
    'Content-Type': 'application/json',
    'x-publora-key': os.environ['PUBLORA_API_KEY']
}

# List connections
connections = requests.get(f'{BASE_URL}/platform-connections', headers=headers).json()

# Create post
post = requests.post(f'{BASE_URL}/create-post', headers=headers, json={
    'content': 'Your post content',
    'platforms': ['twitter-123456789'],
    'scheduledTime': '2026-03-01T14:00:00.000Z',  # Optional
}).json()

# Get post status
status = requests.get(f'{BASE_URL}/get-post/{post_group_id}', headers=headers).json()

# Delete post
requests.delete(f'{BASE_URL}/delete-post/{post_group_id}', headers=headers)
```

## Cursor Composer Workflows

### Workflow 1: Build a Social Media Scheduler

Open Cursor Composer (Cmd+I / Ctrl+I) and use:

```text
Build a social media scheduling feature for my app using Publora API.

Requirements:
1. Form to compose posts with character count
2. Platform selector from connected accounts
3. Date/time picker for scheduling
4. Submit handler that calls Publora API
5. Success/error feedback

Use: [React/Vue/Svelte] with [TypeScript/JavaScript]
API Key is in environment variable PUBLORA_API_KEY
```

### Workflow 2: Add Analytics Dashboard

```text
Create a LinkedIn analytics dashboard using Publora API.

Requirements:
1. Fetch post statistics from /linkedin-post-statistics
2. Display impressions, clicks, likes, comments, shares
3. Calculate engagement rate
4. Show comparison between posts
5. Add date range filter

Platform connection ID: linkedin-ABC123DEF
```

### Workflow 3: Bulk Import Tool

```text
Build a CSV import tool for scheduling posts via Publora.

CSV format:
content,platforms,scheduled_time,media_url

Requirements:
1. File upload and CSV parsing
2. Validation before import
3. Progress indicator
4. Rate limiting (200ms between requests)
5. Summary of successful/failed imports
```

## Debugging with Cursor

### Common Issues and Fixes

When Cursor generates code with issues, use these prompts:

```text
# Fix authentication errors
"The Publora API returns 401. Check that x-publora-key header is set correctly."

# Fix scheduling errors
"Publora returns 'Invalid scheduled time'. Convert scheduledTime to ISO 8601 UTC format."

# Fix platform ID errors
"Publora returns 'Invalid platform ID'. Platform IDs should be format: platform-id (e.g., twitter-123456789)"

# Handle partial failures
"The post shows 'partially_published' status. Check each platform post status individually."
```

### Debug Helper Function

```typescript
// Add this to your project for easier debugging with Cursor
async function debugPubloraRequest(endpoint: string, options: RequestInit = {}) {
  console.log('Publora Request:', {
    url: `https://api.publora.com/api/v1${endpoint}`,
    method: options.method || 'GET',
    body: options.body ? JSON.parse(options.body as string) : undefined,
  });

  const response = await fetch(`https://api.publora.com/api/v1${endpoint}`, {
    ...options,
    headers: {
      'Content-Type': 'application/json',
      'x-publora-key': process.env.PUBLORA_API_KEY!,
      ...options.headers,
    },
  });

  const data = await response.json();

  console.log('Publora Response:', {
    status: response.status,
    ok: response.ok,
    data,
  });

  if (!response.ok) {
    throw new Error(`Publora API error: ${data.error || data.message}`);
  }

  return data;
}
```

## Type Definitions for Better Autocomplete

Add these types to get better Cursor suggestions:

```typescript
// types/publora.d.ts
// Add to your project for Cursor autocomplete

declare namespace Publora {
  interface PlatformConnection {
    platformId: string;
    platform: 'twitter' | 'linkedin' | 'instagram' | 'threads' | 'tiktok' | 'youtube' | 'facebook' | 'bluesky' | 'mastodon' | 'telegram';
    username: string;
    displayName: string;
    profileImageUrl?: string;
  }

  interface CreatePostRequest {
    content: string;
    platforms: string[];
    scheduledTime?: string;
    mediaUrls?: string[];
    mediaKeys?: string[];
    status?: 'draft' | 'scheduled';
    platformSettings?: {
      instagram?: { videoType?: 'REELS' | 'STORIES' };
      tiktok?: { disableDuet?: boolean; disableStitch?: boolean };
      telegram?: { parseMode?: 'HTML' | 'MarkdownV2' };
    };
  }

  interface Post {
    platform: string;
    platformConnectionId: string;
    status: 'draft' | 'scheduled' | 'pending' | 'processing' | 'published' | 'failed';
    publishedUrl?: string;
    error?: string;
  }

  interface PostGroup {
    postGroupId: string;
    content: string;
    status: string;
    scheduledTime?: string;
    posts: Post[];
  }

  interface CreatePostResponse {
    success: boolean;
    postGroupId: string;
    posts: Post[];
  }

  interface LinkedInStats {
    impressions: number;
    clicks: number;
    likes: number;
    comments: number;
    shares: number;
    engagementRate?: number;
  }
}
```

## Context7 Integration

Publora docs are indexed on Context7, which means AI assistants can access up-to-date documentation automatically.

### Using with Cursor + Context7 MCP

If you have Context7 MCP server configured in Cursor:

```text
@context7 How do I schedule a post to multiple platforms with Publora API?
```

```text
@context7 Show me how to upload an image and create a post with Publora
```

```text
@context7 What are the rate limits for Publora API?
```

### Configuring Context7 MCP in Cursor

```json
// .cursor/mcp.json
{
  "mcpServers": {
    "context7": {
      "command": "npx",
      "args": ["-y", "@anthropic/context7-mcp"]
    }
  }
}
```

## Best Practices for AI-Assisted Development

### 1. Provide Context First

Before asking Cursor to write Publora code, share relevant context:

```text
I'm building a social media scheduler using:
- Next.js 14 with App Router
- Publora API for posting
- PostgreSQL for storing post history

My PUBLORA_API_KEY is in .env.local

Now, create an API route to schedule a post...
```

### 2. Iterate Incrementally

Start with simple requests and build up:

```text
Step 1: "Create a function to fetch Publora connections"
Step 2: "Add error handling to the connections function"
Step 3: "Create a React hook that uses this function"
Step 4: "Add caching to reduce API calls"
```

### 3. Ask for Explanations

When Cursor generates code, ask for clarification:

```text
"Explain why you used a 200ms delay between API calls"
"What happens if the scheduledTime is in the past?"
"How does the partial failure handling work?"
```

### 4. Request Tests

Always ask for tests alongside implementation:

```text
"Create unit tests for the Publora service class"
"Add integration tests for the create post endpoint"
"Write mocks for Publora API responses"
```

## Example Test Mocks

```typescript
// __mocks__/publora.ts
// Use with Cursor-generated tests

export const mockConnections = [
  {
    platformId: 'twitter-123456789',
    platform: 'twitter',
    username: '@testuser',
    displayName: 'Test User',
  },
  {
    platformId: 'linkedin-ABC123DEF',
    platform: 'linkedin',
    username: 'Test User',
    displayName: 'Test User',
  },
];

export const mockCreatePostResponse = {
  success: true,
  postGroupId: 'pg_test123',
  posts: [
    { platform: 'twitter', platformConnectionId: 'twitter-123456789', status: 'scheduled' },
    { platform: 'linkedin', platformConnectionId: 'linkedin-ABC123DEF', status: 'scheduled' },
  ],
};

export const mockPostGroup = {
  postGroupId: 'pg_test123',
  content: 'Test post content',
  status: 'published',
  posts: [
    {
      platform: 'twitter',
      status: 'published',
      publishedUrl: 'https://twitter.com/testuser/status/123',
    },
    {
      platform: 'linkedin',
      status: 'published',
      publishedUrl: 'https://linkedin.com/post/123',
    },
  ],
};

export const mockLinkedInStats = {
  impressions: 1250,
  clicks: 45,
  likes: 28,
  comments: 5,
  shares: 3,
  engagementRate: 2.88,
};
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
