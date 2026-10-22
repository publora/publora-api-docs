# Agency publishing formats and settings

Agency client posts use the Company API at `https://api.publora.com/api/v1/company`
and the `company_*` MCP tools. Discover available formats with
`GET /api/v1/company/platforms` or `company_context` action `platforms` before using
these settings. This extension is documented ahead of deployment; the returned
registry is authoritative for your environment.

Use `company_context` to choose an accessible Company and Workspace, then
`company_channels` to copy each exact `channelKey`. API requests use
`x-publora-key` and the selected `X-Publora-Workspace-Id`; MCP commands take
`workspaceId`. Permissions follow the caller's current Workspace role.
Personal post tools and personal upload endpoints do not access Agency posts.

## Save platform settings

Create a draft with `POST /api/v1/company/post-groups`; update it with
`PATCH /api/v1/company/post-groups/:postGroupId`. Send JSON and a stable
`Idempotency-Key`: choose a unique key for each new intended change and keep it
only for retries of the identical request. The MCP equivalent is `company_save_post` with
`command.action` set to `create` or `update` and the fields inside `command.draft`.

`platformSettings` replaces the complete settings map. Preserve settings for the
other selected platforms when changing one. Only selected platforms may carry
settings; unknown fields and nested properties are rejected. Drafts may be
incomplete; scheduling validates the full saved post and ready uploaded media.

| Platform | Company settings | Rules |
| --- | --- | --- |
| Instagram | `videoType`: `REELS` or `STORIES` | A Story requires exactly one image or one video. Captions are omitted for Stories. Image albums remain feed posts. |
| Threads | `replyControl`: `everyone`, `accounts_you_follow`, `mentioned_only` or empty | Empty uses the provider default. Reply controls do not enable post chains. |
| LinkedIn | `repostEnabled`, `repostParentUrn`, `repostVisibility` | Use a canonical `urn:li:share:…` or `urn:li:ugcPost:…`. Commentary is optional and media is forbidden. Visibility is `PUBLIC`, `CONNECTIONS` or empty; Pages require `PUBLIC`. |
| TikTok | `commercialContent`, `brandOrganic`, `brandedContent` | All are booleans. Commercial content requires your brand, a paid partnership, or both. Paid partnerships cannot use `SELF_ONLY`. Existing creator privacy and interaction options still apply. |
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
setting `repostEnabled:false` with a nonempty parent is invalid. For TikTok,
a true disclosure category infers `commercialContent:true` when that flag is
omitted; an explicit false flag with a true category is invalid.

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

MCP uses `company_media` action `reserve` with `purpose:"youtube_thumbnail"`, then
HTTP PUT, `company_media` action `complete`, and `company_save_post` action
`update`. All arguments belong inside `command`.

The thumbnail is auxiliary: it is excluded from ordinary video/image attachments.
`GET /api/v1/company/post-groups/:postGroupId/media` includes the selected
`youtubeThumbnail` preview when present. Delete an auxiliary asset with
`DELETE /api/v1/company/post-groups/:postGroupId/media/:mediaFileId` and `{}`;
deleting the selected asset also clears its reference. Ordinary uploads omit
`purpose` and keep their existing contract. Recovering failed channels into a
new draft clears the source thumbnail reference; upload/select it on the new draft.

## Validation, approvals and retries

Publishing-affecting settings and the selected thumbnail are included in client
review. Editing the Story placement, reply audience, repost, disclosure, tags,
playlist or thumbnail invalidates existing approval. Finish uploads and edits
before requesting a new review. Approval does not schedule a post automatically.

| Response | Meaning | Action |
| --- | --- | --- |
| `400 AGENCY_CORE_CONTENT_INPUT_INVALID` | Unknown field, invalid settings shape, wrong playlist channel or untrusted thumbnail reference | Correct the draft and use a new key for the changed request. |
| `422 AGENCY_CORE_MEDIA_TYPE_MISMATCH` | Invalid reservation or uploaded thumbnail metadata | Use a JPEG/PNG within the thumbnail requirements; follow the returned media state before retrying. |
| `403 AGENCY_CORE_ACCESS_DENIED` | Workspace or role access is unavailable | Refresh Company context and select an accessible Workspace. |
| `409` Company content/media conflict | Draft or upload state changed, or the retry body differs | Refetch the current state; retry an uncertain operation with the identical body and key. |

After a lost response, retain the same workspace, resource IDs, body and
idempotency key. Never send Company requests through personal tools as a fallback.
Instagram Reels covers/`shareToFeed`, Company webhooks, Twitter replies/quotes,
Threads chains and YouTube `categoryId` are outside this extension.

See [Agency first comments](agency-first-comments.md) and the
[Agency MCP tools](../mcp/tools-reference.md#agency-tools-12).
