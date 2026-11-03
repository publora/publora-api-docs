# Agency LinkedIn Page reshares

Use this flow when employees reshare a post published by their company's LinkedIn
Page from their own profiles, each after their own approval. The Page and the
employee profiles are connected channels of the same Agency Workspace. Each
employee gets a separate draft and a separate review link; no Publora account or
Agency seat is needed to approve.

Company API paths are relative to `https://api.publora.com/api/v1/company`. Send
your own `x-publora-key` and the selected `X-Publora-Workspace-Id` on every
request. Mutations need a unique `Idempotency-Key` for each intended operation;
retry an uncertain request with the identical body and key. MCP clients use the
`company_*` tools with top-level `workspaceId`.

1. Subscribe a Workspace webhook to `approval.decided`, `post.published` and
   `post.failed` before requesting reviews. See [Agency webhooks](agency-webhooks.md).
2. Find the source post on the connected Company Page.
3. Create one draft per employee profile with the repost settings and the source snapshot.
4. Request a review for each draft, optionally with a publication time.
5. Distribute each review link privately and track decisions and publication results.

## Find a Company Page post

`POST /linkedin-page-posts` lists published posts authored by one connected
Company Page. It is read-only: it creates no draft, review or publication. The
MCP equivalent is `company_list_linkedin_page_posts`.

```bash
curl -X POST 'https://api.publora.com/api/v1/company/linkedin-page-posts' \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: $WORKSPACE_ID" \
  -H 'Content-Type: application/json' \
  -d '{"connectionId":"64b000000000000000000301","start":0,"count":10,"sortBy":"LAST_MODIFIED"}'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `connectionId` | string | Yes | The Company Page connection's database ID from Company `/connections` (`id` in `company_list_channels`), not a channel key, provider ID or personal connection |
| `start` | integer | No | Offset from 0 to 2147483647, default 0. Pass the previous `paging.nextStart` |
| `count` | integer | No | Page size from 1 to 100, default 10 |
| `sortBy` | string | No | `LAST_MODIFIED` (default) or `CREATED` |

No other body fields are accepted. The author is derived from the authorized
connection: you cannot select another organization, author or URL.

```json
{
  "success":true,
  "data":{
    "connectionId":"64b000000000000000000301",
    "authorUrn":"urn:li:organization:2414183",
    "items":[{
      "postedId":"urn:li:share:7520484550762942467",
      "permalink":"https://www.linkedin.com/feed/update/urn:li:share:7520484550762942467/",
      "commentary":"Company announcement",
      "publishedAt":"2026-10-28T10:00:00.000Z",
      "lastModifiedAt":"2026-10-28T10:00:00.000Z",
      "visibility":"PUBLIC",
      "feedDistribution":"MAIN_FEED",
      "isReshareDisabledByAuthor":false,
      "reshareParentUrn":null,
      "sourceSnapshot":{
        "postedId":"urn:li:share:7520484550762942467",
        "commentary":"Company announcement",
        "authorName":"Company Page",
        "publishedAt":"2026-10-28T10:00:00.000Z",
        "capturedAt":"2026-11-03T12:00:00.000Z"
      }
    }],
    "paging":{"start":0,"count":10,"nextStart":10}
  }
}
```

- `postedId` is always the listed post's own canonical `urn:li:share:…` or
  `urn:li:ugcPost:…` ID. Use it as the repost parent. `reshareParentUrn` is only
  metadata about a post that is itself a reshare; do not use it instead.
- Follow `paging.nextStart` with the same connection, `count` and `sortBy` until it
  is `null`. A short or even empty page can still have a next page. Pagination is
  LinkedIn's live offset, so deduplicate by `postedId` while posts change.
- Only published posts with canonical IDs are returned. Activity IDs are omitted
  and arbitrary LinkedIn URLs are not resolved.
- `commentary` is a preview of at most 30,000 characters. Timestamps, visibility,
  feed distribution, the reshare-disabled flag and the parent are `null` when
  LinkedIn does not provide them. The list can include sponsored posts. Check
  visibility, distribution and `isReshareDisabledByAuthor` before choosing a
  source; a missing flag does not guarantee that LinkedIn accepts the reshare.
- The original can change or disappear after this read. Publication still runs the
  normal LinkedIn checks.

Reading requires internal content view access in the Workspace and a Company Page
connection with a usable token. Organization posts require the Page's existing
`r_organization_social` permission.

| Status/code | Meaning | Recovery |
| --- | --- | --- |
| 400 `LINKEDIN_INVALID_REQUEST` | Unknown field or invalid `start`, `count` or `sortBy` | Correct the request. |
| 400 `LINKEDIN_COMPANY_PAGE_REQUIRED` | The connection is a personal profile, not a Company Page | Use the Page connection's ID. |
| 401 `LINKEDIN_TOKEN_EXPIRED` / `LINKEDIN_RECONNECT_REQUIRED` | The Page connection needs to be reconnected | Reconnect the Page, then retry. |
| 403 `AGENCY_CORE_ACCESS_DENIED` | No current access to this Workspace's content | Refresh context and use an accessible Workspace. |
| 404 `AGENCY_CORE_CONNECTION_NOT_FOUND` | No such LinkedIn connection in the selected Workspace | Refresh the Workspace's connections. |
| 429 `LINKEDIN_RATE_LIMITED` | LinkedIn quota or cooldown | Honor `Retry-After`. |
| 502 `LINKEDIN_INVALID_RESPONSE` | LinkedIn returned an unusable page or continuation | Retry later from the same `start`. |
| 503 `COMPANY_LINKEDIN_UNAVAILABLE` | Temporarily unavailable | Retry later. |

## Create one draft per employee

Create a separate draft for each employee with exactly one target: that employee's
personal LinkedIn channel key (`linkedin-<platformId>` from the Workspace's
connections). Keep the source Page out of the targets. Commentary is optional and
media is forbidden on a repost. See
[Agency platform settings](agency-platform-settings.md) for the repost fields.

```json
{
  "platforms":["linkedin-EMPLOYEE_PLATFORM_ID"],
  "content":"",
  "platformSettings":{
    "linkedin":{
      "repostEnabled":true,
      "repostParentUrn":"urn:li:share:7520484550762942467",
      "repostVisibility":"PUBLIC"
    }
  },
  "repostSourceSnapshot":{
    "postedId":"urn:li:share:7520484550762942467",
    "commentary":"Company announcement",
    "authorName":"Company Page",
    "publishedAt":"2026-10-28T10:00:00.000Z",
    "capturedAt":"2026-11-03T12:00:00.000Z"
  }
}
```

Send it with `POST /post-groups` or update an existing draft with
`PATCH /post-groups/:postGroupId`. MCP clients put the same fields inside `draft`
of `company_create_post` / `company_update_post`.

Optional top-level `repostSourceSnapshot` shows the original post's text, author
and capture time inside the client review. Copy the finder item's `sourceSnapshot`
verbatim:

| Field | Type | Rules |
| --- | --- | --- |
| `postedId` | string | Required. Must equal `platformSettings.linkedin.repostParentUrn` |
| `commentary` | string | Required, at most 30,000 characters (may be empty) |
| `authorName` | string or null | Optional, at most 300 characters |
| `publishedAt` | string or null | Optional exact UTC ISO timestamp from the finder |
| `capturedAt` | string | Required exact UTC ISO timestamp from the finder |

Unknown fields and avatar URLs are rejected. `null` clears the snapshot. It is a
descriptive preview supplied by the draft author, not live LinkedIn data or a
publishing permission, and it does not change what is published. Changing or
clearing it invalidates an existing client approval. Changing the repost parent
or removing the LinkedIn target clears an older snapshot automatically. Save the
snapshot in your stored request plan and resend it unchanged on retries instead
of calling the finder again. Without a snapshot, the review shows the link to the
original on LinkedIn.

`GET /post-groups/:postGroupId?includeRepostSource=true` (MCP: `company_get_post`
with `includeRepostSource: true`) adds `group.repostSourceSnapshot` when a matching
snapshot is saved. Without the parameter the response is unchanged.

## Request reviews

`POST /post-groups/:postGroupId/approval` starts a new review round for the saved
draft and returns its review link (`reviewUrl`). MCP:
`company_request_post_approval`. Finish edits and media uploads first.

```bash
curl -X POST "https://api.publora.com/api/v1/company/post-groups/$POST_GROUP_ID/approval" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: $WORKSPACE_ID" \
  -H "Idempotency-Key: $REVIEW_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"managerComment":"Please review before Tuesday","scheduledTime":"2030-12-10T09:00:00.000Z"}'
```

| Field | Type | Description |
| --- | --- | --- |
| `managerComment` | string | Optional note for the client, at most 5,000 characters |
| `scheduledTime` | string or null | Optional future UTC instant. When set, approval schedules this exact version through the normal scheduler. Omit or `null` keeps manual publication |

- With a future `scheduledTime`, the client's approval and the scheduling commit
  together, with the same checks as a manual schedule. If scheduling validation
  fails, the approval is not saved (`409 COMPANY_APPROVAL_SCHEDULING_FAILED`, or
  `503 COMPANY_APPROVAL_SCHEDULING_UNAVAILABLE` for a temporary failure); fix the
  post or channel, then the client approves again.
- If the time passes before the client approves, the approval is saved and the post
  stays a draft. It is never published immediately instead. Schedule it explicitly
  or request a new round with a new time.
- To change the time of a pending review, request a new round. After approval,
  use the normal reschedule and unschedule operations.
- Clients can approve, decline with an optional comment, or request changes with a
  required comment. Declines and change requests never schedule. Editing text,
  channels, media, publishing settings or the source snapshot makes the round
  outdated; request a new one.

Treat each `reviewUrl` as a bearer credential: deliver it only to its employee and
keep it out of logs and public exports. Store the mapping of employee, channel,
draft and review ID in your application.

## Track decisions and results

Approval and publication are separate states. An `approval.decided` event reports
`approved`, `declined` or `changes_requested` for one review round and carries a
stable `id` for deduplication; for a timed round it includes the publication state
at decision time. `post.published` and `post.failed` report the actual per-channel
outcome later. Deduplicate decisions by envelope `id` and publication events by
`event` + `postId`. A late event for an older round must not replace the current
round. See [Agency webhooks](agency-webhooks.md#approval-decisions).

To recover state after a missed delivery, read:

- `GET /post-groups/:postGroupId/approval` for the current round, decision and
  feedback (MCP: `company_get_post_approval`).
- `GET /post-groups/:postGroupId` for scheduling and per-channel publication
  results (MCP: `company_get_post`).

Clients that understand declined reviews send
`Accept: application/vnd.publora.company-approval.v2+json, application/json` on
approval reads and on list, calendar and detail reads with `includeApproval=true`.
Without it, list and detail responses omit the approval details of a declined
post, and a dedicated approval read of a declined or timed round returns
`409 COMPANY_APPROVAL_CLIENT_UPDATE_REQUIRED`. The MCP tools send the v2 header.

See also [Company MCP tools](../mcp/tools-reference.md#company-tools-49) and
[LinkedIn reshare](../endpoints/linkedin-reshare.md) for personal accounts.
