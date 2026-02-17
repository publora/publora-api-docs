# Next.js Integration

Build social media features into your Next.js app with Publora API.

## Installation

```bash
npm install next
```

No additional dependencies needed - uses native `fetch`.

## Environment Setup

```bash
# .env.local
PUBLORA_API_KEY=sk_your_api_key_here
```

## API Route Handler (App Router)

### Create Post Endpoint

```typescript
// app/api/social/post/route.ts
import { NextRequest, NextResponse } from 'next/server';

const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY!;
const BASE_URL = 'https://api.publora.com/api/v1';

export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const { content, platforms, scheduledTime } = body;

    if (!content || !platforms?.length) {
      return NextResponse.json(
        { error: 'Content and platforms are required' },
        { status: 400 }
      );
    }

    const response = await fetch(`${BASE_URL}/create-post`, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': PUBLORA_API_KEY,
      },
      body: JSON.stringify({
        content,
        platforms,
        scheduledTime,
      }),
    });

    const data = await response.json();

    if (!response.ok) {
      return NextResponse.json(
        { error: data.error || data.message },
        { status: response.status }
      );
    }

    return NextResponse.json(data);
  } catch (error) {
    console.error('Publora API error:', error);
    return NextResponse.json(
      { error: 'Failed to create post' },
      { status: 500 }
    );
  }
}
```

### Get Connections Endpoint

```typescript
// app/api/social/connections/route.ts
import { NextResponse } from 'next/server';

const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY!;
const BASE_URL = 'https://api.publora.com/api/v1';

export async function GET() {
  try {
    const response = await fetch(`${BASE_URL}/platform-connections`, {
      headers: {
        'x-publora-key': PUBLORA_API_KEY,
      },
      next: { revalidate: 60 }, // Cache for 60 seconds
    });

    const data = await response.json();

    if (!response.ok) {
      return NextResponse.json(
        { error: data.error || data.message },
        { status: response.status }
      );
    }

    return NextResponse.json(data);
  } catch (error) {
    console.error('Publora API error:', error);
    return NextResponse.json(
      { error: 'Failed to fetch connections' },
      { status: 500 }
    );
  }
}
```

### Get Post Status Endpoint

```typescript
// app/api/social/post/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server';

const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY!;
const BASE_URL = 'https://api.publora.com/api/v1';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const response = await fetch(`${BASE_URL}/get-post/${params.id}`, {
      headers: {
        'x-publora-key': PUBLORA_API_KEY,
      },
    });

    const data = await response.json();

    if (!response.ok) {
      return NextResponse.json(
        { error: data.error || data.message },
        { status: response.status }
      );
    }

    return NextResponse.json(data);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to fetch post' },
      { status: 500 }
    );
  }
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const response = await fetch(`${BASE_URL}/delete-post/${params.id}`, {
      method: 'DELETE',
      headers: {
        'x-publora-key': PUBLORA_API_KEY,
      },
    });

    const data = await response.json();

    if (!response.ok) {
      return NextResponse.json(
        { error: data.error || data.message },
        { status: response.status }
      );
    }

    return NextResponse.json(data);
  } catch (error) {
    return NextResponse.json(
      { error: 'Failed to delete post' },
      { status: 500 }
    );
  }
}
```

## Publora Service Class

```typescript
// lib/publora.ts
const PUBLORA_API_KEY = process.env.PUBLORA_API_KEY!;
const BASE_URL = 'https://api.publora.com/api/v1';

export interface PlatformConnection {
  platformId: string;
  platform: string;
  username: string;
  displayName: string;
}

export interface CreatePostRequest {
  content: string;
  platforms: string[];
  scheduledTime?: string;
}

export interface PostGroup {
  postGroupId: string;
  content: string;
  status: string;
  posts: Array<{
    platform: string;
    status: string;
    publishedUrl?: string;
    error?: string;
  }>;
}

class PubloraService {
  private async request<T>(
    endpoint: string,
    options: RequestInit = {}
  ): Promise<T> {
    const response = await fetch(`${BASE_URL}${endpoint}`, {
      ...options,
      headers: {
        'Content-Type': 'application/json',
        'x-publora-key': PUBLORA_API_KEY,
        ...options.headers,
      },
    });

    const data = await response.json();

    if (!response.ok) {
      throw new Error(data.error || data.message || 'API request failed');
    }

    return data;
  }

  async getConnections(): Promise<PlatformConnection[]> {
    const data = await this.request<{ connections: PlatformConnection[] }>(
      '/platform-connections'
    );
    return data.connections;
  }

  async createPost(request: CreatePostRequest): Promise<{ postGroupId: string }> {
    return this.request('/create-post', {
      method: 'POST',
      body: JSON.stringify(request),
    });
  }

  async getPost(postGroupId: string): Promise<PostGroup> {
    return this.request(`/get-post/${postGroupId}`);
  }

  async deletePost(postGroupId: string): Promise<{ success: boolean }> {
    return this.request(`/delete-post/${postGroupId}`, {
      method: 'DELETE',
    });
  }

  async getUploadUrl(fileName: string, contentType: string, postGroupId: string) {
    return this.request<{ uploadUrl: string; fileUrl: string; mediaId: string }>(
      '/get-upload-url',
      {
        method: 'POST',
        body: JSON.stringify({ fileName, contentType, postGroupId }),
      }
    );
  }
}

export const publora = new PubloraService();
```

## React Components

### Social Post Form

```tsx
// components/SocialPostForm.tsx
'use client';

import { useState, useEffect } from 'react';

interface Connection {
  platformId: string;
  platform: string;
  username: string;
}

export default function SocialPostForm() {
  const [content, setContent] = useState('');
  const [selectedPlatforms, setSelectedPlatforms] = useState<string[]>([]);
  const [connections, setConnections] = useState<Connection[]>([]);
  const [scheduledTime, setScheduledTime] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [result, setResult] = useState<{ success?: boolean; postGroupId?: string; error?: string } | null>(null);

  useEffect(() => {
    fetch('/api/social/connections')
      .then((res) => res.json())
      .then((data) => setConnections(data.connections || []))
      .catch(console.error);
  }, []);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsLoading(true);
    setResult(null);

    try {
      const response = await fetch('/api/social/post', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          content,
          platforms: selectedPlatforms,
          scheduledTime: scheduledTime || undefined,
        }),
      });

      const data = await response.json();

      if (response.ok) {
        setResult({ success: true, postGroupId: data.postGroupId });
        setContent('');
        setSelectedPlatforms([]);
        setScheduledTime('');
      } else {
        setResult({ success: false, error: data.error });
      }
    } catch (error) {
      setResult({ success: false, error: 'Failed to create post' });
    } finally {
      setIsLoading(false);
    }
  };

  const togglePlatform = (platformId: string) => {
    setSelectedPlatforms((prev) =>
      prev.includes(platformId)
        ? prev.filter((p) => p !== platformId)
        : [...prev, platformId]
    );
  };

  return (
    <form onSubmit={handleSubmit} className="space-y-4">
      <div>
        <label className="block text-sm font-medium mb-1">Content</label>
        <textarea
          value={content}
          onChange={(e) => setContent(e.target.value)}
          className="w-full p-2 border rounded"
          rows={4}
          required
        />
        <p className="text-sm text-gray-500 mt-1">{content.length} characters</p>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Platforms</label>
        <div className="flex flex-wrap gap-2">
          {connections.map((conn) => (
            <button
              key={conn.platformId}
              type="button"
              onClick={() => togglePlatform(conn.platformId)}
              className={`px-3 py-1 rounded ${
                selectedPlatforms.includes(conn.platformId)
                  ? 'bg-blue-500 text-white'
                  : 'bg-gray-200'
              }`}
            >
              {conn.platform}: {conn.username}
            </button>
          ))}
        </div>
      </div>

      <div>
        <label className="block text-sm font-medium mb-1">Schedule (optional)</label>
        <input
          type="datetime-local"
          value={scheduledTime}
          onChange={(e) => setScheduledTime(e.target.value)}
          className="p-2 border rounded"
        />
      </div>

      <button
        type="submit"
        disabled={isLoading || !content || selectedPlatforms.length === 0}
        className="px-4 py-2 bg-blue-500 text-white rounded disabled:opacity-50"
      >
        {isLoading ? 'Posting...' : 'Schedule Post'}
      </button>

      {result && (
        <div className={`p-3 rounded ${result.success ? 'bg-green-100' : 'bg-red-100'}`}>
          {result.success
            ? `Post created: ${result.postGroupId}`
            : `Error: ${result.error}`}
        </div>
      )}
    </form>
  );
}
```

### Post Status Component

```tsx
// components/PostStatus.tsx
'use client';

import { useState, useEffect } from 'react';

interface PostStatusProps {
  postGroupId: string;
}

interface Post {
  platform: string;
  status: string;
  publishedUrl?: string;
  error?: string;
}

interface PostGroup {
  postGroupId: string;
  status: string;
  posts: Post[];
}

export default function PostStatus({ postGroupId }: PostStatusProps) {
  const [postGroup, setPostGroup] = useState<PostGroup | null>(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    const fetchStatus = async () => {
      try {
        const response = await fetch(`/api/social/post/${postGroupId}`);
        const data = await response.json();
        setPostGroup(data);
      } catch (error) {
        console.error('Failed to fetch post status:', error);
      } finally {
        setIsLoading(false);
      }
    };

    fetchStatus();

    // Poll every 10 seconds if still processing
    const interval = setInterval(() => {
      if (postGroup?.status === 'scheduled' || postGroup?.status === 'processing') {
        fetchStatus();
      }
    }, 10000);

    return () => clearInterval(interval);
  }, [postGroupId, postGroup?.status]);

  if (isLoading) return <div>Loading...</div>;
  if (!postGroup) return <div>Post not found</div>;

  const statusColors: Record<string, string> = {
    published: 'bg-green-100 text-green-800',
    scheduled: 'bg-blue-100 text-blue-800',
    processing: 'bg-yellow-100 text-yellow-800',
    failed: 'bg-red-100 text-red-800',
    partially_published: 'bg-orange-100 text-orange-800',
  };

  return (
    <div className="border rounded p-4">
      <div className="flex items-center justify-between mb-4">
        <h3 className="font-medium">Post: {postGroupId}</h3>
        <span className={`px-2 py-1 rounded text-sm ${statusColors[postGroup.status] || 'bg-gray-100'}`}>
          {postGroup.status}
        </span>
      </div>

      <div className="space-y-2">
        {postGroup.posts.map((post, index) => (
          <div key={index} className="flex items-center justify-between p-2 bg-gray-50 rounded">
            <span className="font-medium">{post.platform}</span>
            <div className="text-right">
              <span className={`text-sm ${post.status === 'published' ? 'text-green-600' : post.status === 'failed' ? 'text-red-600' : 'text-gray-600'}`}>
                {post.status}
              </span>
              {post.publishedUrl && (
                <a href={post.publishedUrl} target="_blank" rel="noopener noreferrer" className="ml-2 text-blue-500 text-sm">
                  View
                </a>
              )}
              {post.error && <p className="text-red-500 text-xs">{post.error}</p>}
            </div>
          </div>
        ))}
      </div>
    </div>
  );
}
```

## Server Actions (Next.js 14+)

```typescript
// app/actions/social.ts
'use server';

import { publora } from '@/lib/publora';
import { revalidatePath } from 'next/cache';

export async function createSocialPost(formData: FormData) {
  const content = formData.get('content') as string;
  const platforms = formData.getAll('platforms') as string[];
  const scheduledTime = formData.get('scheduledTime') as string | null;

  if (!content || platforms.length === 0) {
    return { error: 'Content and at least one platform are required' };
  }

  try {
    const result = await publora.createPost({
      content,
      platforms,
      scheduledTime: scheduledTime || undefined,
    });

    revalidatePath('/dashboard');
    return { success: true, postGroupId: result.postGroupId };
  } catch (error) {
    return { error: error instanceof Error ? error.message : 'Failed to create post' };
  }
}

export async function deleteSocialPost(postGroupId: string) {
  try {
    await publora.deletePost(postGroupId);
    revalidatePath('/dashboard');
    return { success: true };
  } catch (error) {
    return { error: error instanceof Error ? error.message : 'Failed to delete post' };
  }
}

export async function getConnections() {
  try {
    return await publora.getConnections();
  } catch (error) {
    console.error('Failed to get connections:', error);
    return [];
  }
}
```

## Server Component with Data Fetching

```tsx
// app/dashboard/page.tsx
import { publora } from '@/lib/publora';
import SocialPostForm from '@/components/SocialPostForm';

export default async function DashboardPage() {
  const connections = await publora.getConnections();

  return (
    <div className="container mx-auto p-4">
      <h1 className="text-2xl font-bold mb-6">Social Media Dashboard</h1>

      <div className="grid gap-6 md:grid-cols-2">
        <div>
          <h2 className="text-lg font-semibold mb-4">Connected Accounts</h2>
          <div className="space-y-2">
            {connections.map((conn) => (
              <div key={conn.platformId} className="p-3 border rounded flex justify-between">
                <span className="font-medium capitalize">{conn.platform}</span>
                <span className="text-gray-600">{conn.username}</span>
              </div>
            ))}
          </div>
        </div>

        <div>
          <h2 className="text-lg font-semibold mb-4">Create Post</h2>
          <SocialPostForm />
        </div>
      </div>
    </div>
  );
}
```

## Middleware for API Protection

```typescript
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  // Protect social API routes
  if (request.nextUrl.pathname.startsWith('/api/social')) {
    // Add your authentication logic here
    // For example, check for a session cookie or JWT

    const authHeader = request.headers.get('authorization');

    if (!authHeader) {
      return NextResponse.json(
        { error: 'Authentication required' },
        { status: 401 }
      );
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: '/api/social/:path*',
};
```

## Error Handling Hook

```typescript
// hooks/usePublora.ts
'use client';

import { useState, useCallback } from 'react';

interface UsePubloraOptions {
  onSuccess?: (data: any) => void;
  onError?: (error: string) => void;
}

export function usePublora(options: UsePubloraOptions = {}) {
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState<string | null>(null);

  const createPost = useCallback(
    async (content: string, platforms: string[], scheduledTime?: string) => {
      setIsLoading(true);
      setError(null);

      try {
        const response = await fetch('/api/social/post', {
          method: 'POST',
          headers: { 'Content-Type': 'application/json' },
          body: JSON.stringify({ content, platforms, scheduledTime }),
        });

        const data = await response.json();

        if (!response.ok) {
          throw new Error(data.error || 'Failed to create post');
        }

        options.onSuccess?.(data);
        return data;
      } catch (err) {
        const message = err instanceof Error ? err.message : 'Unknown error';
        setError(message);
        options.onError?.(message);
        throw err;
      } finally {
        setIsLoading(false);
      }
    },
    [options]
  );

  const getConnections = useCallback(async () => {
    setIsLoading(true);
    setError(null);

    try {
      const response = await fetch('/api/social/connections');
      const data = await response.json();

      if (!response.ok) {
        throw new Error(data.error || 'Failed to fetch connections');
      }

      return data.connections;
    } catch (err) {
      const message = err instanceof Error ? err.message : 'Unknown error';
      setError(message);
      throw err;
    } finally {
      setIsLoading(false);
    }
  }, []);

  return {
    createPost,
    getConnections,
    isLoading,
    error,
  };
}
```

---

*[Publora](https://publora.com) — Affordable social media API starting at $5.40/month*
