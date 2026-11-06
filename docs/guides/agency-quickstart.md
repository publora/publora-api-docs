# Agency quickstart (Company API and MCP)

Agencies manage client posts through the **Company API** and the `company_*` MCP
tools. Each client is a **Workspace** with its own connected channels. Every call
runs with the signed-in member's own role, and Publora checks permissions in each
Workspace. The personal API (`/create-post`, MCP `create_post`) is a separate
surface: never retry a refused Company command through it.

This page walks through one post end to end: context → channels → draft → media →
client review → schedule → result and recovery.

## Access

| | REST | MCP |
|---|---|---|
| Endpoint | `https://api.publora.com/api/v1/company` | `https://mcp.publora.com/mcp` |
| Authentication | `x-publora-key: <PUBLORA_API_KEY>` | `Authorization: Bearer <PUBLORA_API_KEY>` or the member's Publora login |
| Client Workspace | `X-Publora-Workspace-Id` header on post, channel, media and review calls | `workspaceId` argument |
| Retry key | `Idempotency-Key` header | `idempotencyKey` argument |
| Draft fields | JSON request body | inside the `draft` argument |
| Channel key | build `platform + "-" + platformId` from each channel | returned as `channelKey` |
| Review reads | send the approval `Accept` header (step 5) | sent by the tool |
| Validation details | add `?includeValidationDetails=true` to schedule calls | always requested |

The key is the member's own Publora API key. A personal key never grants Owner
authority. MCP clients read current tool schemas from `tools/list`; the
[MCP tools reference](../mcp/tools-reference.md#company-tools-52) lists all
Company tools.

## 1. Pick the Agency and the client Workspace

```bash
curl https://api.publora.com/api/v1/company/contexts/v2 \
  -H "x-publora-key: $PUBLORA_API_KEY"
```

MCP: `company_list_contexts`.

`data.accounts[]` lists each Company the member belongs to, with `id`, `kind`,
`accountRole`, `capabilities` and `workspaces[]`. Each Workspace has `id`, `name`,
`roleSlug` and `capabilities` such as `canWriteDrafts`, `canUploadMedia` and
`canScheduleOrPublish`. `data.defaultContext` is set only when exactly one
Account/Workspace pair exists.

Confirm with the user which client they mean. Never guess a Workspace or reuse one
from another conversation. Check the capability before each step instead of
retrying a refused command.

## 2. List the client's channels

```bash
curl https://api.publora.com/api/v1/company/connections \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: 64b000000000000000000201"
```

MCP: `company_list_channels` with `workspaceId`.

Channels are in `data.items[]` (`[]` when none are connected), each with `id`,
`platform`, `platformId`, `username`, `displayName` and `status`. A channel key is
`platform + "-" + platformId`, for example `linkedin-abc123`; MCP adds it as
`channelKey`. Copy it exactly. A channel with `status: "reconnect_required"` cannot
publish until the client reconnects it. `GET /platforms` (MCP
`company_get_platforms`) returns the formats and settings Company posts support.

## 3. Create the draft

```bash
curl -X POST https://api.publora.com/api/v1/company/post-groups \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: 64b000000000000000000201" \
  -H "Idempotency-Key: acme-launch-draft-1" \
  -H "Content-Type: application/json" \
  -d '{"content": "Our new product is live.", "platforms": ["linkedin-abc123"]}'
```

The response is `201` with `data.groupId`, the post group ID used in every later
step. A draft takes no `scheduledTime` or `status`; `platforms: []` is a valid
channel-less draft. Edit it with `PATCH /post-groups/{postGroupId}`.

MCP `company_create_post` wraps the same fields in `draft`:

```json
{
  "workspaceId": "64b000000000000000000201",
  "idempotencyKey": "acme-launch-draft-1",
  "draft": { "content": "Our new product is live.", "platforms": ["linkedin-abc123"] }
}
```

Edit with `company_update_post` (`workspaceId`, `postGroupId`, `idempotencyKey`,
`draft`). Platform options go in `platformSettings` ([Agency platform
settings](agency-platform-settings.md)); one first comment goes in `firstComment`
([Agency first comment](agency-first-comments.md)).

## 4. Attach media

Instagram, TikTok and YouTube need media before scheduling.

1. Reserve: `POST /post-groups/{postGroupId}/media/reservations` with an
   `Idempotency-Key` and `{"fileName", "contentType", "sizeBytes"}` (MCP
   `company_reserve_media`). The response has `mediaFileId`, `uploadUrl`,
   `uploadHeaders` and `expiresAt`.
2. Upload the exact bytes with HTTP `PUT` to `uploadUrl`, sending **all**
   `uploadHeaders` and **no** Publora key.
3. Complete: `POST /post-groups/{postGroupId}/media/{mediaFileId}/complete` with
   `{}` and no idempotency key (MCP `company_complete_media`).

`409 AGENCY_CORE_MEDIA_UPLOAD_PENDING` means the upload is still being inspected:
retry **complete** after the `Retry-After` header, do not reserve again. After
`AGENCY_CORE_MEDIA_RESERVATION_EXPIRED`, delete the reservation and reserve again.

## 5. Client review (optional)

Request a review when the client approves posts before publishing:

```bash
curl -X POST https://api.publora.com/api/v1/company/post-groups/$POST_GROUP_ID/approval \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: 64b000000000000000000201" \
  -H "Idempotency-Key: acme-launch-review-1" \
  -H "Content-Type: application/json" \
  -d '{"managerComment": "Launch post for Thursday", "scheduledTime": "2026-11-20T09:00:00.000Z"}'
```

MCP: `company_request_post_approval`.

- The response contains `data.reviewUrl`, a private review page valid until
  `data.expiresAt` (30 days). Publora does not send it to the client: deliver it
  yourself, treat it as a bearer credential and keep it out of logs.
- When the client decides, Publora emails the user who requested the review. If
  `notificationStatus` is `failed`, retry with `POST .../approval/notification`
  (MCP `company_retry_approval_notification`).
- Save all edits and complete media first. The command needs
  `canScheduleOrPublish`; both fields are optional.
- With a future `scheduledTime` (UTC, with milliseconds), the post is **scheduled
  automatically once approved**: do not schedule it again. If that time passes
  before the decision, the post stays a draft and needs a new time.
- Without `scheduledTime`, the approved post stays a draft until you schedule it.
- While a review is requested, the post can be scheduled only with the client's
  current approval. Changing text, channels, media or settings invalidates it;
  request a new round. `DELETE .../approval` (MCP `company_cancel_post_approval`)
  removes the requirement; use it only when the user asks.
- REST reads of the review (`GET .../approval`, or post reads with
  `includeApproval=true`) need
  `Accept: application/vnd.publora.company-approval.v2+json, application/json`.
  Without it, `GET .../approval` returns
  `409 COMPANY_APPROVAL_CLIENT_UPDATE_REQUIRED` for declined and timed reviews,
  and post reads leave out a declined review. MCP sends the header itself.

## 6. Schedule

```bash
curl -X POST "https://api.publora.com/api/v1/company/post-groups/$POST_GROUP_ID/schedule?includeValidationDetails=true" \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: 64b000000000000000000201" \
  -H "Idempotency-Key: acme-launch-schedule-1" \
  -H "Content-Type: application/json" \
  -d '{"scheduledTime": "2026-11-20T09:00:00.000Z"}'
```

MCP: `company_schedule_post`. The body is exactly `{"scheduledTime"}` in UTC. To
publish now, schedule at the current UTC time and keep that same time on retries.
Confirm text, channels, client and time with the user first. Change the time with
`/reschedule` or return the post to a draft with `/unschedule` (MCP
`company_reschedule_post`, `company_unschedule_post`).

## 7. Check the result and recover failures

Scheduling only queues the post; delivery is asynchronous.
`GET /post-groups/{postGroupId}` (MCP `company_get_post`) returns the group in
`data.group` and each target in `data.posts[]`, each with its `status`.

For failed targets, `POST /post-groups/{postGroupId}/recover` with `{}` and an
`Idempotency-Key` (MCP `company_recover_post`) creates a **new draft**, returned as
`data.groupId`, with only the failed targets and their media. Published parts are
never sent again. Inspect the new draft before scheduling it.

## Retries

These commands need an idempotency key: draft create, update and delete; schedule,
reschedule, unschedule and recover; media reservation; requesting or cancelling a
client review; channel disconnect; Workspace create, rename and archive; ownership
transfer; webhook changes. Completing or deleting media, retrying a review
notification and connection-link commands take no key.

After a lost response, retry with the **same key, body and time**, then read the
resource. `409 AGENCY_CORE_IDEMPOTENCY_IN_FLIGHT` means the first request is still
running: wait for `Retry-After` and retry unchanged.
`409 AGENCY_CORE_IDEMPOTENCY_KEY_CONFLICT` means the key was used for a different
request. A new intended operation gets a new key.

## Errors

Company commands return `{"success": false, "code", "message"}`; API-key
authentication failures return `{"error"}`. `402 COMPANY_BILLING_REQUIRED` means
the Agency's billing does not allow the command: tell the user instead of retrying.
See [Error handling](error-handling.md) for HTTP status handling.

## Corporate accounts

A Corporate account (`kind: "team"`, `presentationMode: "people"`) has
`workspaces: []`: it has no client Workspaces, channels or reviews in the Company
API. Corporate employees publish their own posts through the personal API or MCP
tools.

## Related guides

- [Agency platform settings](agency-platform-settings.md): formats, media rules and per-platform options
- [Agency first comment](agency-first-comments.md)
- [Agency LinkedIn Page reshares](agency-linkedin-reshares.md)
- [Agency LinkedIn Page reads](agency-linkedin-page-reads.md)
- [Agency webhooks](agency-webhooks.md)
- [MCP tools reference](../mcp/tools-reference.md#company-tools-52)
