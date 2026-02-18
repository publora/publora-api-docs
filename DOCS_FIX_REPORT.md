# Publora API Documentation Fix Report

**Date:** 2026-02-17
**Commits:** `d31c750`, `bb62573` (pushed to `main`)
**Files changed:** 35 files, ~1,300 insertions, ~1,450 deletions

---

## Problem

The API documentation had two competing vocabularies. The **endpoint reference pages** (`/endpoints/*`) used correct field names matching the actual backend API (`apiRoute.js`), but **guide pages**, **platform docs**, **code examples**, and **integration docs** used outdated/incorrect field names throughout.

This means any developer reading the guides or code examples would write code that doesn't work against the real API.

---

## What Was Wrong vs What Is Correct

| Category | Wrong (was in docs) | Correct (matches API) |
|----------|--------------------|-----------------------|
| **Post content field** | `text` | `content` |
| **Platforms field** | `platformIds` | `platforms` |
| **Create post endpoint** | `POST /post-groups` | `POST /create-post` |
| **Get post endpoint** | `GET /post-groups/{id}` | `GET /get-post/{id}` |
| **Update post endpoint** | `PUT /post-groups/{id}` | `PUT /update-post/{id}` |
| **Delete post endpoint** | `DELETE /post-groups/{id}` | `DELETE /delete-post/{id}` |
| **Connections endpoint** | `GET /connections` | `GET /platform-connections` |
| **Connection ID field** | `id` | `platformId` |
| **Media in create-post** | `mediaUrls: [...]` | Does not exist (use upload workflow) |
| **Media keys in create-post** | `mediaKeys: [...]` | Does not exist (auto-attaches via postGroupId) |
| **Upload param: MIME type** | `mimeType` | `contentType` |
| **Upload param: post ref** | (missing) | `postGroupId` (required) |
| **Upload response: URL** | `presignedUrl` | `uploadUrl` |
| **Upload response: key** | `mediaKey` | `fileUrl` + `mediaId` |
| **Upload workflow order** | Get URL -> Upload -> Create post | Create post -> Get URL (with postGroupId) -> Upload |
| **LinkedIn: account param** | `platformConnectionId` | `platformId` |
| **LinkedIn: post param** | `postUrn` | `postedId` |
| **LinkedIn: query param** | (missing) | `queryTypes: "ALL"` |
| **LinkedIn: response shape** | Flat: `impressions`, `likes`, `comments`, `shares` | Nested: `metrics.IMPRESSION`, `metrics.REACTION`, `metrics.COMMENT`, `metrics.RESHARE`, `metrics.MEMBERS_REACHED` |

---

## Files Changed

### Guides (8 files)
| File | Key Fixes |
|------|-----------|
| `docs/guides/scheduling.md` | `/post-groups` -> `/create-post`, `text` -> `content`, `platformIds` -> `platforms` |
| `docs/guides/analytics.md` | LinkedIn stats: `platformConnectionId` -> `platformId`, `postUrn` -> `postedId`, flat metrics -> nested `metrics.*` |
| `docs/guides/media-uploads.md` | Removed `mediaUrls`/`mediaKeys`/`presignedUrl`/`mediaKey`, rewrote upload workflow with correct order and field names |
| `docs/guides/cross-platform.md` | `/post-groups` -> `/create-post`, `text` -> `content`, `platformIds` -> `platforms`, `/connections` -> `/platform-connections` |
| `docs/guides/bulk-scheduling.md` | Removed `mediaUrls`, fixed error field |
| `docs/guides/workspace.md` | `/post-groups` -> `/create-post`, `text` -> `content`, `/connections` -> `/platform-connections` |
| `docs/guides/error-handling.md` | `/post-groups` -> `/create-post`/`/get-post`, `text` -> `content`, `platformIds` -> `platforms` |
| `docs/guides/cursor-ai.md` | Removed `mediaUrls`/`mediaKeys` from types, fixed LinkedIn stats interface, `platformConnectionId` -> `platformId` |

### Platform Docs (10 files)
| File | Key Fixes |
|------|-----------|
| `docs/platforms/x-twitter.md` | Removed `mediaUrls` from 4 code examples, added media upload workflow note |
| `docs/platforms/instagram.md` | Removed `mediaUrls` from 12 code examples (image, carousel, reel), added media upload note |
| `docs/platforms/youtube.md` | Removed `mediaUrls` from 12 code examples, added media upload note |
| `docs/platforms/tiktok.md` | Removed `mediaUrls` from 8 code examples, added media upload note |
| `docs/platforms/telegram.md` | Removed `mediaUrls` from 8 code examples, added media upload note |
| `docs/platforms/linkedin.md` | Removed `mediaUrls` from 4 code examples, added media upload note |
| `docs/platforms/facebook.md` | Removed `mediaUrls` from 4 code examples, added media upload note |
| `docs/platforms/threads.md` | Removed `mediaUrls` from 4 code examples, added media upload note |
| `docs/platforms/mastodon.md` | Removed `mediaUrls` from 8 code examples, added media upload note |
| `docs/platforms/bluesky.md` | Removed `mediaUrls` from 8 code examples, updated `altTexts` mapping description, added media upload note |

### Language Examples (10 files)
| File | Key Fixes |
|------|-----------|
| `docs/examples/javascript/quick-start.md` | `c.id` -> `c.platformId`, `platformIds` -> `platforms` |
| `docs/examples/javascript/media-upload.md` | Complete rewrite: correct 3-step upload workflow, `contentType` not `mimeType`, `fileUrl`/`mediaId` not `mediaKey` |
| `docs/examples/javascript/scheduling-workflows.md` | Removed `mediaUrls` |
| `docs/examples/python/quick-start.md` | `c['id']` -> `c['platformId']` |
| `docs/examples/python/bulk-csv-import.md` | Removed 3 `mediaUrls` references |
| `docs/examples/typescript/quick-start.md` | Fixed types (`mediaUrls`, `mediaKeys`, `mediaKey` -> `mediaId`), upload workflow, LinkedIn analytics |
| `docs/examples/php/quick-start.md` | `platformConnectionId` -> `platformId`, `postUrn` -> `postedId`, `mediaKey` -> `mediaId`, removed `mediaUrls`/`mediaKeys` |
| `docs/examples/ruby/quick-start.md` | Same fixes as PHP |
| `docs/examples/go/quick-start.md` | Same fixes as PHP, updated Go struct types |
| `docs/examples/curl/all-endpoints.md` | Fixed connections response, media workflow, upload params/response, LinkedIn params/response, bash script |

### Framework & No-Code Examples (6 files)
| File | Key Fixes |
|------|-----------|
| `docs/examples/frameworks/nextjs.md` | Removed `mediaUrls`/`mediaKeys` from types, fixed `getUploadUrl` params |
| `docs/examples/frameworks/laravel.md` | Fixed LinkedIn stats params, upload params, removed `media_urls` validation |
| `docs/examples/frameworks/django.md` | Fixed `create_post`, `get_upload_url`, `get_linkedin_stats` params, removed `media_urls` from serializer |
| `docs/examples/no-code/n8n-integration.md` | Reordered upload workflow, `mimeType` -> `contentType`, removed `mediaKeys` step |
| `docs/examples/no-code/make-integration.md` | Reordered upload workflow, `mimeType` -> `contentType`, removed `mediaKeys` step |
| `docs/examples/no-code/zapier-integration.md` | No changes needed (was already correct) |

### Endpoint Docs (1 file)
| File | Key Fixes |
|------|-----------|
| `docs/endpoints/upload-media.md` | Fixed upload workflow step order (create post first, then get upload URL) |
| `docs/endpoints/linkedin-statistics.md` | Fixed in previous session (account-statistics section) |

---

## Live Web Docs Status (docs.publora.com)

After checking the live site on 2026-02-17:

### Already Correct (no redeploy needed for these)
- `/getting-started` - uses `content`, `platforms`, correct response shapes
- `/authentication` - correct header names and error responses
- `/endpoints/create-post` - correct schema
- `/endpoints/get-post` - correct schema with `platformId`, `postedId`
- `/endpoints/update-post` - correct schema
- `/endpoints/delete-post` - correct schema
- `/endpoints/platform-connections` - correct schema with `platformId`
- `/endpoints/linkedin-statistics` - correct schema with `platformId`, `postedId`, nested `metrics`

### Will Be Fixed After Redeploy
All guide pages, platform docs, and code examples listed above. The changes are committed and pushed to `main`.

---

## Action Required

1. **Redeploy docs.publora.com** from the latest `main` branch to pick up all 35 file changes
2. **Verify** by spot-checking these pages after deploy:
   - `/guides/scheduling` - should show `/create-post`, `content`, `platforms`
   - `/guides/media-uploads` - should show correct 3-step workflow (create post first)
   - `/platforms/instagram` - should NOT have `mediaUrls` in any code example
   - `/examples/typescript/quick-start` - should show `contentType`, `fileUrl`, `mediaId`
3. **If using Context7**: After redeploy, the docs will be re-indexed automatically on next crawl. The corrected field names will then be served to AI coding assistants.

---

## Verification

Final grep across all 48 docs files confirms **zero remaining instances** of:
- `mediaUrls` / `mediaKeys`
- `platformConnectionId` / `postUrn`
- `/post-groups` endpoint
- `text:` as an API field (vs `content:`)
- `presignedUrl` / `mediaKey` (old upload response fields)
- `mimeType` as an API parameter (vs `contentType`)
