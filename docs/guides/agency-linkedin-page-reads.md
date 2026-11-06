# Agency LinkedIn Page reads

Read known posts, comments and reactions of a LinkedIn Company Page connected to
an Agency Workspace. All three operations are read-only: they create no draft,
review, comment or reaction, and need no `Idempotency-Key`.

| Operation | REST | MCP |
| --- | --- | --- |
| Read up to 50 posts by ID, optionally with engagement summaries | `POST /linkedin-page-posts/batch` | `company_get_linkedin_page_posts_batch` |
| List top-level comments on one post | `POST /linkedin-page-posts/comments` | `company_list_linkedin_page_post_comments` |
| List reactions on one post | `POST /linkedin-page-posts/reactions` | `company_list_linkedin_page_post_reactions` |

Company API paths are relative to `https://api.publora.com/api/v1/company`. Send
your own `x-publora-key` and the selected `X-Publora-Workspace-Id` on every
request. MCP clients pass the same fields as top-level arguments together with
`workspaceId`.

`connectionId` is the Company Page connection's database ID from Company
`/connections` (`id` in `company_list_channels`), not a channel key, provider ID or
personal connection. Post IDs are canonical `urn:li:share:…` or
`urn:li:ugcPost:…` URNs, for example the `postedId` values returned by
[`POST /linkedin-page-posts`](agency-linkedin-reshares.md#find-a-company-page-post).
Activity IDs and LinkedIn URLs are not accepted. Unknown body fields are rejected.

## Permissions

| Read | Workspace access | LinkedIn permissions of the Page connection |
| --- | --- | --- |
| Posts (batch without engagement) | Internal content view (Manager, Publisher, Contributor or Viewer) | `r_organization_social` |
| Engagement (batch with `includeEngagement`, comments, reactions) | Internal content view and analytics view (Manager, Publisher or Contributor) | `r_organization_social` and `r_organization_social_feed` |

The author is always the Page behind `connectionId`: you cannot read another
organization's posts or select an author.

If the Page connection lacks a required LinkedIn permission, the request returns
`403 LINKEDIN_INSUFFICIENT_SCOPE`. Batch reads also need a Page connection
authorized through Publora's current LinkedIn app; an older connection returns
`403 LINKEDIN_BATCH_UNAVAILABLE` before any LinkedIn call. In both cases the error
`message` contains the recovery steps. Usually it asks you to reconnect the Page:
open **Channels** in the Workspace, choose **Copy reconnect link** on the Page's
card (or ask a Workspace manager for the link), have a LinkedIn Page admin
authorize the same Page, then retry. If the message asks the Workspace manager to
contact Publora support instead, reconnecting will not help. MCP clients receive
the same message; show it to the user and wait for the reconnect before retrying.

## Read posts by ID

```bash
curl -X POST 'https://api.publora.com/api/v1/company/linkedin-page-posts/batch' \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: $WORKSPACE_ID" \
  -H 'Content-Type: application/json' \
  -d '{"connectionId":"64b000000000000000000301","postedIds":["urn:li:share:7520484550762942467","urn:li:ugcPost:7520484551723372544"],"includeEngagement":true}'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `connectionId` | string | Yes | The Company Page connection's database ID |
| `postedIds` | string[] | Yes | 1–50 share or ugcPost URNs. Duplicates are read once |
| `includeEngagement` | boolean | No | Default `false`. `true` adds comment and reaction summaries and needs analytics view access |

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
        "capturedAt":"2026-11-05T12:00:00.000Z"
      },
      "engagement":{
        "commentsState":"OPEN",
        "commentCount":12,
        "topLevelCommentCount":9,
        "reactions":{"LIKE":40,"PRAISE":3}
      }
    }],
    "errors":[{
      "postedId":"urn:li:ugcPost:7520484551723372544",
      "source":"post",
      "status":404,
      "code":"LINKEDIN_NOT_FOUND"
    }]
  }
}
```

- Successful posts appear in `items` in request order, with the same fields and
  `sourceSnapshot` as the [Page post list](agency-linkedin-reshares.md#find-a-company-page-post).
- With `includeEngagement`, each returned post also has `engagement`, or `null`
  when its summary is unavailable. `commentsState` (`OPEN`, `CLOSED`,
  `PROCESSING` or `DELETED`), `commentCount` and `topLevelCommentCount` are
  `null` when LinkedIn does not provide them. `reactions` maps each LinkedIn
  reaction type to its count.
- Each failed post is listed once per source in `errors` with `postedId`,
  `source` (`post` or `engagement`), the numeric `status` and a `code`:
  `LINKEDIN_TOKEN_EXPIRED` (401), `LINKEDIN_ACCESS_DENIED` (403),
  `LINKEDIN_NOT_FOUND` (404), `LINKEDIN_RATE_LIMITED` (429) or
  `LINKEDIN_ITEM_UNAVAILABLE`. A missing, unpublished or other author's post is
  omitted from `items`. A verified post whose engagement failed stays in `items`.
- The request uses one LinkedIn Posts batch read, plus one Social Metadata batch
  read for the verified posts when `includeEngagement` is `true`.
- If LinkedIn throttles some posts, the response stays HTTP 200 with the other
  results, `data.retryAfter` and a `Retry-After` header in seconds. If LinkedIn
  throttles every post in either batch read, the request returns
  `429 LINKEDIN_RATE_LIMITED`.

## List comments or reactions

Send the same body to `/linkedin-page-posts/comments` or
`/linkedin-page-posts/reactions`.

```bash
curl -X POST 'https://api.publora.com/api/v1/company/linkedin-page-posts/comments' \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: $WORKSPACE_ID" \
  -H 'Content-Type: application/json' \
  -d '{"connectionId":"64b000000000000000000301","postedId":"urn:li:share:7520484550762942467","start":0,"count":10}'
```

| Field | Type | Required | Description |
| --- | --- | --- | --- |
| `connectionId` | string | Yes | The Company Page connection's database ID |
| `postedId` | string | Yes | One share or ugcPost URN authored by this Page |
| `start` | integer | No | Offset from 0 to 2147483647, default 0. Pass the previous `paging.nextStart` |
| `count` | integer | No | Page size from 1 to 100, default 10 |

Publora first verifies with the Page's token that the exact post exists, is
published and is authored by the Page, then reads one page of its top-level
comments or reactions. A normal page uses these two LinkedIn calls within one
request deadline. A missing, unpublished or other author's post returns
`404 LINKEDIN_NOT_FOUND`.

```json
{
  "success":true,
  "data":{
    "connectionId":"64b000000000000000000301",
    "authorUrn":"urn:li:organization:2414183",
    "postedId":"urn:li:share:7520484550762942467",
    "items":[{
      "commentUrn":"urn:li:comment:(urn:li:activity:7520484551000000000,7520490000000000000)",
      "parentCommentUrn":null,
      "actorUrn":"urn:li:person:AbC123xyz",
      "message":"Congratulations to the team!",
      "createdAt":"2026-10-28T11:02:00.000Z",
      "lastModifiedAt":"2026-10-28T11:02:00.000Z"
    }],
    "paging":{"start":0,"count":10,"nextStart":null}
  }
}
```

- A comment item has `commentUrn` (LinkedIn's canonical comment URN, returned
  verbatim), `parentCommentUrn`, `actorUrn`, `message` (at most 30,000
  characters), `createdAt` and `lastModifiedAt`.
- A reaction item has `actorUrn`, `reactionType`, `createdAt` and
  `lastModifiedAt`. Reactions are ordered newest first. Current types are `LIKE`,
  `PRAISE`, `EMPATHY`, `INTEREST`, `APPRECIATION` and `ENTERTAINMENT`; older
  reactions can be `MAYBE`.
- Timestamps are UTC ISO strings. Missing text, actor, parent or time values are
  `null`. `actorUrn` is a LinkedIn person or organization URN; person URNs are
  specific to the LinkedIn app that issued them, so do not match them against IDs
  from another app.
- Follow `paging.nextStart` as the next `start` with the same `count`,
  connection and post until it is `null`. A full final page can be followed by an
  empty one. The list can change between pages: deduplicate comments by
  `commentUrn` and reactions by a non-null `actorUrn`.
- A comments read past the last comment returns an empty page after Publora
  confirms the comment count with LinkedIn; if LinkedIn cannot confirm it, the
  request returns `404 LINKEDIN_NOT_FOUND`.

## Errors

Errors use the Company envelope `{ "success": false, "code": "...", "message": "..." }`.

| Status/code | Meaning | Recovery |
| --- | --- | --- |
| 400 `LINKEDIN_INVALID_REQUEST` | Unknown field, invalid post IDs, `includeEngagement`, `start` or `count` | Correct the request. |
| 400 `LINKEDIN_COMPANY_PAGE_REQUIRED` | The connection is a personal profile, not a Company Page | Use the Page connection's ID. |
| 401 `LINKEDIN_TOKEN_EXPIRED` / `LINKEDIN_RECONNECT_REQUIRED` | The Page connection needs to be reconnected | Reconnect the Page, then retry. |
| 403 `AGENCY_CORE_ACCESS_DENIED` | No current access to this Workspace's content, or engagement requested without analytics view access | Refresh context; use an accessible Workspace and role. |
| 403 `LINKEDIN_BATCH_UNAVAILABLE` | Batch only: the Page connection was authorized through an older LinkedIn app | Follow the steps in `message`. |
| 403 `LINKEDIN_INSUFFICIENT_SCOPE` | The Page connection lacks a required LinkedIn read permission | Follow the steps in `message`. |
| 403 `LINKEDIN_ACCESS_DENIED` | LinkedIn refused access to the resource | Check the Page admin's access, then retry. |
| 404 `AGENCY_CORE_CONNECTION_NOT_FOUND` | No such LinkedIn connection in the selected Workspace | Refresh the Workspace's connections. |
| 404 `LINKEDIN_NOT_FOUND` | Comments/reactions: the post is missing, unpublished or not authored by this Page | Use a `postedId` from the Page's post list. |
| 429 `LINKEDIN_RATE_LIMITED` | LinkedIn quota or cooldown | Honor `Retry-After`; do not switch Workspace to bypass it. |
| 502 `LINKEDIN_INVALID_RESPONSE` | LinkedIn returned a response Publora could not verify | Retry later with the same request. |
| 5xx `LINKEDIN_TEMPORARILY_UNAVAILABLE` | The LinkedIn read failed | Retry later. |
| 503 `COMPANY_LINKEDIN_UNAVAILABLE` | Temporarily unavailable | Retry later. |
| 504 `LINKEDIN_REQUEST_TIMEOUT` | The LinkedIn read deadline elapsed | Retry with backoff. |

See also [Agency LinkedIn Page reshares](agency-linkedin-reshares.md) and
[Company MCP tools](../mcp/tools-reference.md#company-tools-52).
