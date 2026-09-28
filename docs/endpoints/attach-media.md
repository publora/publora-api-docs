# Attach Media

Attach one image or video to an existing **draft** or **scheduled** post by passing a file reference: an HTTPS download URL plus a stable file ID. Publora downloads the file right away, checks the actual bytes and attaches it. You don't need a presigned `PUT` or a `complete-media` call.

This endpoint backs the MCP `attach_media` tool. ChatGPT fills that tool's `file` argument from a file in the conversation through OpenAI's `openai/fileParams`; see [MCP Tools Reference](../mcp/tools-reference.md). Any API client can call the endpoint directly.

The post is **left in draft**. This endpoint always sets the status to `draft`, so a post that was `scheduled` goes back to `draft`. (By contrast, `mediaUrls` on `update-post` keeps a scheduled post scheduled and re-validates it.) To publish it, schedule it again with [`update-post`](./update-post.md).

## Endpoint

```
POST https://api.publora.com/api/v1/attach-media/:postGroupId
```

## Headers

| Header | Required | Description |
|--------|----------|-------------|
| `x-publora-key` | Yes | Your API key |
| `x-publora-user-id` | No | Managed user ID (workspace only) |
| `x-publora-client` | No | Client identifier (e.g., `"mcp"`) |
| `Content-Type` | Yes | `application/json` |
| `Idempotency-Key` | No | Recommended. With it, a retry replays the first result instead of attaching the file a second time. See [Idempotency](#idempotency). The MCP tool always sends one. |

## Path Parameters

| Parameter | Type | Description |
|-----------|------|-------------|
| `postGroupId` | string | ID of an existing draft or scheduled post (24 hexadecimal characters). If you need a new one, first create a draft with [`create-post`](./create-post.md) and leave out `scheduledTime`. |

## Request Body

The body can contain only `file` and `fileName`. Any other top-level field, such as `status`, `scheduledTime` or `mediaUrls`, returns `400 INVALID_MEDIA_FILE`: this endpoint attaches exactly one file and never schedules or publishes.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `file` | object | Yes | The file reference described below. A string is rejected, including a local path such as `/mnt/data/image.png`, base64 data or a bare URL. |
| `fileName` | string | No | A descriptive file name with an extension, 1–255 characters, e.g. `spring-launch-banner.png`. Takes precedence over `file.file_name`. See [File name](#file-name). |

**`file` object.** It must contain only these keys. Any other key is rejected.

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `download_url` | string | Yes | The HTTPS URL Publora downloads the file from. Temporary signed URLs work. It goes through the same checks as [`mediaUrls`](./create-post.md): `https://` only, no embedded credentials, only the default port 443, and no private or loopback hosts. |
| `file_id` | string | Yes | A stable identifier for the file, up to 512 characters. Together with the post ID and the final file name, it lets Publora recognise a retry. See [Idempotency](#idempotency). |
| `mime_type` | string | No | Only a hint. Publora determines the real type from the file's bytes. |
| `file_name` | string | No | The original file name. Used when `fileName` is absent. |

### File name

Publora takes the name from `fileName` if it is present, otherwise from `file.file_name`, and uses `chatgpt-media` only when both are absent. It keeps only the last path segment, trims it, replaces each run of whitespace with `_`, and removes every character other than ASCII letters, digits, `_`, `.` and `-`, so accented and non-Latin letters are dropped. If the result is empty or only dots, the request is rejected, even when the name came from `file.file_name`: an unusable `file.file_name` is not replaced by `chatgpt-media`. Pass an ASCII `fileName` to avoid this.

[`get-post`](./get-post.md) returns the cleaned name as `media[].sourceFileName`. The stored object name and the media URL end with it after a unique prefix, so two files with the same name never overwrite each other. Social networks may rename the file again when they re-host it.

## What happens

1. **Ownership and state checks** run before anything is downloaded. A post that doesn't exist or belongs to another user returns `404`. A published, failed or partially published post returns `400 POST_NOT_EDITABLE`. A post that is publishing, or that already has a live platform post or one with an unknown publish outcome, returns `409 POST_PUBLISH_IN_PROGRESS`.
2. **Rate limit.** Each attempt that gets this far uses one URL from the allowance it shares with `mediaUrls`: 60 URLs per fixed one-hour window. Idempotent replays don't count. Going over it returns `429 MEDIA_URL_RATE_LIMITED` with `Retry-After`.
3. **Download and validation.** The same pipeline as `mediaUrls`: redirects are followed with the same host checks, the type is detected from the bytes, and the file is probed. Accepted formats: JPEG, PNG, GIF, WebP, TIFF and AVIF images up to 25 MB, and MP4, MOV and WebM videos up to 150 MB. Anything else fails without changing the post.
4. **Attach.** The file is appended to the post's existing media with status `ready`, and the post's status is set to `draft`. Publora keeps its own copy, so the temporary download URL isn't needed afterwards.

As with any other media, platform rules such as media count, format and duration for each target are checked when you schedule the post.

## Idempotency

Send an `Idempotency-Key` header whose value stays the same across retries of one attachment. Publora recognises a retry by the post ID, `file_id` and the final file name. `download_url` and `mime_type` are left out, so a retry with a refreshed temporary URL counts as the same attachment.

| Situation | Result |
|---|---|
| Same key, same post, same `file_id` and file name, even with a new `download_url` | The first response is **replayed**. The file is attached only once. |
| Same key, but a different post, `file_id` or file name | `422` `IDEMPOTENCY_KEY_CONFLICT` |
| Same key while the first request is still running | `409` `IDEMPOTENCY_IN_FLIGHT` |
| Same key after the download failed | Processed again. A failed download frees the key, so retry with a fresh file reference. |
| No key | Every call attaches another copy of the file. |

Keys are scoped to the acting user and kept for **24 hours**. A replay returns the earlier result even if you have since removed the file from the post; to attach the same file again within that window, use a new key. If you don't know whether an attachment went through after the window has passed, check [`get-post`](./get-post.md) before retrying.

## Response

The response has the same shape as [`update-post`](./update-post.md#response):

```json
{
  "success": true,
  "message": "Post updated successfully",
  "scheduledTime": null,
  "postGroup": {
    "_id": "507f1f77bcf86cd799439011",
    "status": "draft",
    "content": "Our spring collection is here.",
    "platforms": ["linkedin-ABC123"]
  }
}
```

`postGroup.status` is always `draft`. A post that was scheduled keeps its stored `scheduledTime` as a draft, so the top-level `scheduledTime` and `postGroup.scheduledTime` can be non-null. Unlike `get-upload-url`, this endpoint returns no `postGroupDemoted` flag and sends no [`post.demoted`](./webhooks.md) webhook. `postGroup.status` is the signal.

The response does not include the new media ID. To see the attachment, call [`get-post`](./get-post.md). Its [`media` array](./get-post.md#media-inventory-media) lists the file with `status: "ready"`:

```json
{
  "mediaId": "6a0000000000000000000099",
  "sourceFileName": "spring-launch-banner.png",
  "fileName": "images/1790000000000-0-1a2b3c4d-spring-launch-banner.png",
  "type": "image",
  "mimeType": "image/png",
  "status": "ready",
  "failureReason": null,
  "url": "https://media.publora.com/images/1790000000000-0-1a2b3c4d-spring-launch-banner.png"
}
```

## Examples

### cURL

```bash
POST_GROUP_ID="507f1f77bcf86cd799439011"

curl -sS -X POST "https://api.publora.com/api/v1/attach-media/$POST_GROUP_ID" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: attach-$POST_GROUP_ID-file-abc123" \
  -d '{
    "file": {
      "download_url": "https://files.example.com/download/abc123?signature=TEMPORARY",
      "file_id": "file-abc123",
      "mime_type": "image/png",
      "file_name": "image.png"
    },
    "fileName": "spring-launch-banner.png"
  }'
```

Then, when the post should go out, schedule it:

```bash
FUTURE_TIME=$(date -u -v+1H +%Y-%m-%dT%H:%M:%S.000Z 2>/dev/null || date -u -d '+1 hour' +%Y-%m-%dT%H:%M:%S.000Z)

curl -sS -X PUT "https://api.publora.com/api/v1/update-post/$POST_GROUP_ID" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "Idempotency-Key: schedule-$POST_GROUP_ID-$FUTURE_TIME" \
  -d "{\"status\":\"scheduled\",\"scheduledTime\":\"$FUTURE_TIME\"}"
```

### JavaScript (fetch)

```javascript
const postGroupId = '507f1f77bcf86cd799439011';
const file = {
  download_url: 'https://files.example.com/download/abc123?signature=TEMPORARY',
  file_id: 'file-abc123',
  mime_type: 'image/png'
};

const response = await fetch(`https://api.publora.com/api/v1/attach-media/${postGroupId}`, {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-publora-key': process.env.PUBLORA_API_KEY,
    // One key per logical attachment; reuse it for every retry.
    'Idempotency-Key': `attach-${postGroupId}-${file.file_id}`
  },
  body: JSON.stringify({ file, fileName: 'spring-launch-banner.png' })
});

const result = await response.json();
if (!response.ok) {
  // mediaResults explains a failed download, e.g. MEDIA_URL_HTTP_ERROR for an expired URL.
  throw new Error(`${result.code || result.error}: ${JSON.stringify(result.mediaResults || [])}`);
}
console.log(result.postGroup.status); // "draft"
```

### Python (requests)

```python
import os
import requests

post_group_id = "507f1f77bcf86cd799439011"
file_ref = {
    "download_url": "https://files.example.com/download/abc123?signature=TEMPORARY",
    "file_id": "file-abc123",
}

response = requests.post(
    f"https://api.publora.com/api/v1/attach-media/{post_group_id}",
    headers={
        "x-publora-key": os.environ["PUBLORA_API_KEY"],
        "Idempotency-Key": f"attach-{post_group_id}-{file_ref['file_id']}",
    },
    json={"file": file_ref, "fileName": "spring-launch-banner.png"},
)
response.raise_for_status()
print(response.json()["postGroup"]["status"])  # draft
```

## MCP tool

The MCP `attach_media` tool takes `postGroupId`, `file`, an optional `fileName` and an optional `idempotencyKey`. It calls this endpoint and, when you don't pass a key, derives one from the post ID and `file_id`. The tool declares `_meta["openai/fileParams"]: ["file"]`, so ChatGPT supplies `download_url` and `file_id` itself. See the [MCP Tools Reference](../mcp/tools-reference.md).

## Errors

| Status | Error | Cause |
|--------|-------|-------|
| 400 | `"Only file and fileName are accepted; attaching media leaves the post in draft"` — code `INVALID_MEDIA_FILE` | The body has another top-level field, such as `status` or `scheduledTime`. Nothing was downloaded. |
| 400 | `"file must be a file object with download_url and file_id, not a local path or base64 string"`, `"file.download_url must be a non-empty string"`, `"file only accepts download_url, file_id, mime_type and file_name"` and similar — code `INVALID_MEDIA_FILE` | `file` is missing, isn't an object, lacks a required field, has an unknown key or a non-string value, or `file_id` is over 512 characters |
| 400 | `"Only https:// media URLs are allowed (got http://)"` and other URL-check messages — code `INVALID_MEDIA_FILE` | `download_url` is not a valid URL, is not HTTPS, embeds credentials, uses a non-443 port, or its host is a private or loopback IP address or a `localhost`, `.local` or `.internal` name. Nothing was downloaded. A host name that only resolves to a private address is refused during the download instead, in `mediaResults[]`. |
| 400 | `"fileName must be a non-empty string of at most 255 characters"` / `"fileName must contain a valid file name"` — code `INVALID_MEDIA_FILE` | `fileName` is `null` or not a string, or the name used (`fileName`, else `file.file_name`) is over 255 characters or is empty or only dots after cleaning. An unusable `file.file_name` is not replaced by `chatgpt-media`. |
| 400 | `"Invalid post group ID"` | `postGroupId` is not a valid post ID. Use the 24-character ID from `create-post` or `list-posts`. |
| 400 | `"Media ingestion failed"` with `mediaResults[]` | Download or validation failed. The entry for the URL has `ok: false`, a `code` such as `MEDIA_URL_HTTP_ERROR` (often an expired temporary URL), `MEDIA_URL_TOO_LARGE` or `MEDIA_URL_UNSUPPORTED_FORMAT`, and an `error` message. Nothing was attached. See [`mediaUrls` per-URL codes](../guides/error-codes.md#mediaurls-per-url-codes). |
| 400 | `"Cannot update post: post is currently in {status} status"` — code `POST_NOT_EDITABLE` | The post is published, failed or partially published |
| 400 | `"Invalid platform connection(s): …"` — code `INVALID_PLATFORM_CONNECTION` | The post targets a YouTube connection that is no longer yours. YouTube targets are re-checked when the attachment is saved, and the downloaded file is discarded. The response includes `invalidPlatforms`. |
| 401 | `"API key is required"` / `"Invalid API key"` | Missing or invalid `x-publora-key` |
| 403 | `"API access is not enabled for this account"` / `"MCP access is not enabled for this account"` | The account lacks API access, or MCP access when called through MCP |
| 404 | `"Post group not found"` | No such post, or it belongs to another user |
| 409 | `"Publishing has already started; media cannot be changed. Duplicate the post to recover its content."` — code `POST_PUBLISH_IN_PROGRESS` | The post is publishing, or one of its platform posts is publishing, already live or has an unknown publish outcome. Re-read it with [`get-post`](./get-post.md) before retrying. |
| 409 | code `POST_GROUP_VERSION_CONFLICT` | Another write finished first, and this request changed nothing. Re-read the post and retry. |
| 409 | code `IDEMPOTENCY_IN_FLIGHT` | A request with the same `Idempotency-Key` is still running. Retry the same request shortly. |
| 422 | code `IDEMPOTENCY_KEY_CONFLICT` | The key was already used for a different post, `file_id` or file name |
| 429 | code `MEDIA_URL_RATE_LIMITED` | Over the limit of 60 media URLs per hour. Wait for `Retry-After` seconds. |
| 500 | `"Failed to update post"` | Internal server error |

---

*[Publora](https://publora.com) is built by [Creative Content Crafts, Inc.](https://cccrafts.ai) Need AI-powered content creation for LinkedIn, Threads, and X? Try [Co.Actor](https://co.actor) — the best AI service for authentic thought leadership at scale.*
