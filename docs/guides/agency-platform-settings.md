# Agency publishing formats and settings

Agency client posts use the Company API at `https://api.publora.com/api/v1/company`
and the `company_*` MCP tools. Discover available formats with
`GET /api/v1/company/platforms` or `company_get_platforms` before using
these settings. The returned registry is authoritative
for your environment.

Use `company_list_contexts` and `company_get_workspace_context` to choose an accessible Company and Workspace, then
`company_list_channels` to copy each exact `channelKey`. API requests use
`x-publora-key` and the selected `X-Publora-Workspace-Id`; MCP commands take
`workspaceId`. Permissions follow the caller's current Workspace role.
Personal post tools and personal upload endpoints do not access Agency posts.

## Save platform settings

Create a draft with `POST /api/v1/company/post-groups`; update it with
`PATCH /api/v1/company/post-groups/:postGroupId`. Send JSON and a stable
`Idempotency-Key`: choose a unique key for each new intended change and keep it
only for retries of the identical request. The MCP equivalents are `company_create_post` and
`company_update_post`, with fields inside `draft` and top-level `workspaceId`,
`idempotencyKey` and, for updates, `postGroupId`.

`platformSettings` replaces the complete settings map. Preserve settings for the
other selected platforms when changing one. Only selected platforms may carry
settings; unknown fields and nested properties are rejected. Drafts may be
incomplete; scheduling validates the full saved post and ready uploaded media.

| Platform | Company settings | Rules |
| --- | --- | --- |
| Instagram | `videoType`: `REELS` or `STORIES`, `coverUrl`, `shareToFeed` | A Story requires exactly one image or one video; captions are omitted. Reels accept a public JPEG cover URL and feed sharing. Image albums remain feed posts. |
| Threads | `replyControl`: `everyone`, `accounts_you_follow`, `mentioned_only` or empty | Empty uses the provider default. Reply controls do not enable post chains. |
| LinkedIn | `repostEnabled`, `repostParentUrn`, `repostVisibility` | Use a canonical `urn:li:share:…` or `urn:li:ugcPost:…`. Commentary is optional and media is forbidden. Visibility is `PUBLIC`, `CONNECTIONS` or empty; Pages require `PUBLIC`. |
| TikTok | `commercialContent`, `brandOrganic`, `brandedContent`, `autoAddMusic` | All are booleans. Commercial content requires your brand, a paid partnership, or both. Paid partnerships cannot use `SELF_ONLY`. Automatic music can be enabled for photo Direct Post only. Existing creator privacy and interaction options still apply; scheduling requires `viewerSetting`. |
| YouTube | `tags`, `playlist`, `thumbnail` | Tags are a string array. Playlist and thumbnail references use the exact objects below. |

YouTube tags are trimmed and limited to 500 characters in total, including commas
and quotation marks around tags containing spaces. A playlist is
`{id:"PL…",platformId:"youtube-CHANNEL_ID"}`: select exactly one YouTube channel
and use its complete channel key. IDs are at most 200 characters and channel keys
at most 300. Send `{id:"",platformId:""}` to clear it. Choose a new playlist when
changing the YouTube channel.

A thumbnail is `{mediaId:"24_HEX_MEDIA_ID"}` belonging to this draft and Workspace.
Send `{mediaId:""}` to clear it. External URLs and personal media IDs are rejected.
Playlist addition and thumbnail setting run after video processing and are best
effort: a failure of either does not republish a successfully uploaded video.

Example: an Instagram Story draft, followed by the ordinary Company media upload
flow for a single image or video. Set `STORY_CREATE_KEY` once for this new draft;
when retrying, keep its value and rerun only the curl command:

```bash
STORY_CREATE_KEY="$(uuidgen)"

curl -X POST 'https://api.publora.com/api/v1/company/post-groups' \
  -H 'x-publora-key: sk_YOUR_API_KEY' \
  -H 'X-Publora-Workspace-Id: YOUR_WORKSPACE_ID' \
  -H "Idempotency-Key: $STORY_CREATE_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"platforms":["instagram-YOUR_ACCOUNT_ID"],"platformSettings":{"instagram":{"videoType":"STORIES"}}}'
```

For a Reel, set the cover and feed sharing in the same draft settings map:

```json
{
  "platformSettings":{
    "instagram":{
      "videoType":"REELS",
      "coverUrl":"https://cdn.example.com/covers/reel.jpg",
      "shareToFeed":false
    }
  }
}
```

`coverUrl` is trimmed, limited to 2,048 characters and must be a publicly reachable
HTTP(S) JPEG URL. An empty string clears the custom cover. Extensionless CDN URLs
are accepted; URLs with a known non-JPEG image suffix are rejected. Instagram
fetches the cover when publishing, so keep it accessible until then. The Company
API does not expose the personal cover-upload endpoint; supply your own public
JPEG URL. Omitted `shareToFeed` means `true`; set `false` to publish only as a Reel.
Both options apply only to Reel videos and are ignored for Stories and feed images.

A LinkedIn repost can have empty content and no attachments:

```json
{
  "content":"",
  "platforms":["linkedin-YOUR_ACCOUNT_ID"],
  "platformSettings":{
    "linkedin":{
      "repostEnabled":true,
      "repostParentUrn":"urn:li:share:123456789",
      "repostVisibility":"PUBLIC"
    }
  }
}
```

Supplying a LinkedIn parent without `repostEnabled` infers repost mode. Explicitly
setting `repostEnabled:false` with a nonempty parent, or `repostEnabled:true`
without one, is invalid. For TikTok,
a true disclosure category infers `commercialContent:true` when that flag is
omitted; an explicit false flag with a true category is invalid.

## TikTok automatic photo music

Opt in on a photo draft through REST with:

```json
{"platformSettings":{"tiktok":{"viewerSetting":"PUBLIC_TO_EVERYONE","autoAddMusic":true}}}
```

For `company_create_post`/`company_update_post`, place this settings object inside
`draft`. TikTok will automatically choose the music. You cannot select or preview
a specific track in Publora. Explicit `false` is preserved and sent for photos;
omission preserves existing behavior and does not silently enable music.
Company updates replace the full settings map, so retain any other desired
settings. Draft saving, loading, editing, duplication and scheduling preserve
the value.

Only JSON booleans are accepted; strings, numbers and `null` fail validation.
Enabling the setting for known non-photo media fails validation. Send `false`
to disable music when switching a draft to video. Incomplete drafts without
media are allowed. The publisher sends an explicit value as `post_info.auto_add_music`
only for photo Direct Post and never in video requests. No additional OAuth
scope is needed. See [TikTok automatic photo music](../platforms/tiktok.md#automatic-music-for-photo-posts).

Changing this setting invalidates an existing client approval. Request approval
of the changed draft before scheduling/publishing when review is required.

## Upload an Agency YouTube thumbnail

1. Save a draft with its selected YouTube channel and video.
2. Reserve an auxiliary image at
   `POST /api/v1/company/post-groups/:postGroupId/media/reservations`, using the
   Company headers and a stable `Idempotency-Key`:

   ```json
   {
     "fileName":"thumbnail.png",
     "contentType":"image/png",
     "sizeBytes":24576,
     "type":"image",
     "purpose":"youtube_thumbnail"
   }
   ```

3. PUT the exact bytes to the returned `uploadUrl`, using **all** `uploadHeaders`
   and **no** Publora API key. JPEG or PNG is required, at most 2 MiB and at least
   640×360 pixels. Your YouTube channel must be eligible for custom thumbnails.
4. Complete with
   `POST /api/v1/company/post-groups/:postGroupId/media/:mediaFileId/complete`
   and `{}`. Completion uses the same Company headers and no idempotency key.
   A pending result means retry completion for the same reservation.
5. PATCH the draft's settings, preserving any other desired options:

   ```json
   {
     "platformSettings":{
       "youtube":{
         "privacy":"public",
         "tags":["restaurant","weekly menu"],
         "playlist":{"id":"PL_example","platformId":"youtube-YOUR_CHANNEL_ID"},
         "thumbnail":{"mediaId":"64b000000000000000000501"}
       }
     }
   }
   ```

MCP uses `company_reserve_media` with `purpose:"youtube_thumbnail"`, then
HTTP PUT, `company_complete_media`, and `company_update_post`.
Arguments are top-level; the saved thumbnail reference belongs inside
`draft.platformSettings.youtube.thumbnail.mediaId`.

The thumbnail is auxiliary: it is excluded from ordinary video/image attachments.
`GET /api/v1/company/post-groups/:postGroupId/media` includes the selected
`youtubeThumbnail` preview when present. Delete an auxiliary asset with
`DELETE /api/v1/company/post-groups/:postGroupId/media/:mediaFileId` and `{}`;
deleting the selected asset also clears its reference. Ordinary uploads omit
`purpose` and keep their existing contract. Recovering failed channels into a
new draft clears the source thumbnail reference; upload/select it on the new draft.

## Validation, approvals and retries

Publishing-affecting settings and the selected thumbnail are included in client
review. Editing the Story placement, Reels cover/feed sharing, reply audience, repost, disclosure, TikTok automatic photo music, tags,
playlist or thumbnail invalidates existing approval. Finish uploads and edits
before requesting a new review. Approval does not schedule a post automatically.

| Response | Meaning | Action |
| --- | --- | --- |
| `400 AGENCY_CORE_CONTENT_INPUT_INVALID` | Unknown field, invalid settings shape, wrong playlist channel or untrusted thumbnail reference | Correct the draft and use a new key for the changed request. |
| `422 AGENCY_CORE_MEDIA_TYPE_MISMATCH` | Invalid reservation or uploaded thumbnail metadata | Use a JPEG/PNG within the thumbnail requirements; follow the returned media state before retrying. |
| `422 AGENCY_CORE_MEDIA_SIZE_EXCEEDED` | The file is larger than allowed (2 MiB for a thumbnail) | Reserve a smaller file. |
| `429 AGENCY_CORE_MEDIA_WORKSPACE_QUOTA_EXCEEDED` | Workspace upload budget or pending-upload limit reached | Respect `Retry-After`; complete or delete pending uploads. |
| `403 AGENCY_CORE_ACCESS_DENIED` | Workspace or role access is unavailable | Refresh Company context and select an accessible Workspace. |
| `409` Company content/media conflict | Draft or upload state changed, or the retry body differs | Refetch the current state; retry an uncertain operation with the identical body and key. |

After a lost response, retain the same workspace, resource IDs, body and
idempotency key. Never send Company requests through personal tools as a fallback.
Workspace post-event webhooks are described in [Agency webhooks](agency-webhooks.md).
Twitter replies/quotes, Threads chains and YouTube `categoryId` remain outside this extension.

See [Agency first comments](agency-first-comments.md) and the
[Agency MCP tools](../mcp/tools-reference.md#company-tools-48).
