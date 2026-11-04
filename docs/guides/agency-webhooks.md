# Agency Workspace webhooks

Use the Company surface, not the personal `/webhooks` endpoints.

A subscription belongs to one Agency Workspace. Its managers can create,
list, edit, disable, delete and rotate its signing secret. Authority is the
existing Workspace settings permission; Account membership alone is not enough.
The subscription survives removal of the teammate who created it.

## Manage subscriptions

Use your own `x-publora-key` and the selected `X-Publora-Workspace-Id` on every
request. Company API paths are relative to `https://api.publora.com/api/v1`.
All mutations require a unique `Idempotency-Key` for each intended operation.
After an uncertain response retry the identical Workspace, path, body and key.
Do not generate a new key for a retry. Permission is checked again on replay.

| Method | Path | Body |
| --- | --- | --- |
| GET | `/company/webhooks` | None |
| POST | `/company/webhooks` | `{name,url,events,isActive?}` |
| PATCH | `/company/webhooks/:webhookId` | Any nonempty subset of `{name,url,events,isActive}` |
| DELETE | `/company/webhooks/:webhookId` | `{}` |
| POST | `/company/webhooks/:webhookId/regenerate-secret` | `{}` |

Names are trimmed, nonempty and at most 100 characters. URLs are trimmed and at
most 2048 characters, using public HTTP(S). Private/local destinations are refused
and DNS is checked again at delivery. Redirects are not followed. HTTPS is
recommended. Subscribe to one to three of `post.published`, `post.failed` and
`approval.decided`; other events and duplicate event names are rejected. Existing
subscriptions are not expanded automatically. `isActive` defaults to true.
The limit is ten subscriptions per Workspace, including disabled subscriptions.

Clients that manage `approval.decided` send
`Accept: application/vnd.publora.company-webhooks.v2+json` on every Company
webhook read and mutation. Without it, the publication-only JSON shape is kept,
but a request that would create, return or replay a subscription with
`approval.decided` fails with `409 COMPANY_WEBHOOK_CLIENT_UPDATE_REQUIRED`. Once
such a subscription exists in the Workspace, this includes the list. Nothing is
dropped or relabelled. The app and the MCP tools send the v2 header.

Set this key once for this new subscription; reuse it only for identical retries:

```bash
HOOK_CREATE_KEY="$(uuidgen)"
curl -X POST 'https://api.publora.com/api/v1/company/webhooks' \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: $WORKSPACE_ID" \
  -H "Idempotency-Key: $HOOK_CREATE_KEY" \
  -H 'Content-Type: application/json' \
  -d '{"name":"Client publishing","url":"https://receiver.example.com/publora","events":["post.published","post.failed"]}'
```

To also receive client review decisions, add the event and the v2 header:

```bash
HOOK_CREATE_KEY="$(uuidgen)"
curl -X POST 'https://api.publora.com/api/v1/company/webhooks' \
  -H "x-publora-key: $PUBLORA_API_KEY" \
  -H "X-Publora-Workspace-Id: $WORKSPACE_ID" \
  -H "Idempotency-Key: $HOOK_CREATE_KEY" \
  -H 'Accept: application/vnd.publora.company-webhooks.v2+json' \
  -H 'Content-Type: application/json' \
  -d '{"name":"Advocacy results","url":"https://receiver.example.com/publora","events":["approval.decided","post.published","post.failed"]}'
```

List returns `{success:true,data:{accountId,workspaceId,items:[...]}}`.
Create, update and rotation return
`{success:true,data:{accountId,workspaceId,webhook:{...}}}`. Create and rotation
also return sibling `data.secret`, including retries of that exact operation.
Store it securely; normal list/update responses never expose it. Rotating a
secret invalidates the previous secret for subsequent deliveries. Each webhook
DTO has `id`, `name`, `url`, `events`, `isActive`, `failureCount`,
`lastTriggeredAt` (nullable), `createdAt` and `updatedAt`.
Delete returns `{success:true,data:{accountId,workspaceId,webhookId,deleted:true}}`.

In the app, open Workspace settings and use Webhooks. MCP exposes
`company_list_webhooks`, `company_create_webhook`, `company_update_webhook`,
`company_delete_webhook` and `company_regenerate_webhook_secret`. Pass top-level
`workspaceId`, a stable `idempotencyKey` for mutations and `webhookId` for an
existing subscription. Create fields (`name`, `url`, `events`, optional
`isActive`) are top-level; update fields are nested in `changes`. A creation or rotation
secret is displayed only for the completed operation; it is not a list field.

## Events and signature verification

Publication events describe an individual platform post, not completion of the entire
multi-platform group. The destination is selected by the exact Company and
Workspace pair. Company events are never delivered to personal subscriptions,
and personal/token/billing events are never delivered to Company subscriptions.
Inactive Companies and archived Workspaces do not start new deliveries.

```json
{
  "version":"1",
  "event":"post.published",
  "timestamp":"2026-10-22T12:00:00.000Z",
  "data":{
    "accountId":"64b000000000000000000101",
    "workspaceId":"64b000000000000000000201",
    "postId":"64b000000000000000000401",
    "postGroupId":"64b000000000000000000301",
    "platform":"instagram",
    "platformId":"INSTAGRAM_CHANNEL_ID",
    "postedId":"PROVIDER_POST_ID",
    "permalink":"https://www.instagram.com/p/EXAMPLE/",
    "publishedAt":"2026-10-22T12:00:00.000Z"
  }
}
```

`post.failed` has the same Company/Workspace/post/group/platform/channel identity,
plus `error` and `failedAt`, instead of publication result fields. Error data
contains the publishing failure code/message and may identify an affected media
file. A provider may not supply a permalink or external post ID.

Every request carries `X-Publora-Event` and `X-Publora-Signature`. The signature
is the hex HMAC-SHA256 of the **exact raw request body bytes**, using the webhook
secret. Verify it before trusting or processing the body, with a constant-time
comparison. Do not parse and re-stringify JSON before checking the signature.

Publication events reuse the existing best-effort transport: one HTTP attempt per dispatch,
a ten-second timeout, 2xx success only and no automatic HTTP retry queue. Duplicate
dispatches are possible; process receivers idempotently using `event + postId`
(and your stored processing state), not `postGroupId` alone. There is no durable
outbox or exactly-once guarantee. After five consecutive failures, a subscription
is disabled. Re-enable it after correcting its URL/receiver; enabling resets the
failure count. A delivery already started may finish after disable, deletion or
secret rotation.

## Approval decisions

`approval.decided` reports a client's decision on one review round: `approved`,
`declined` or `changes_requested`. Requesting, expiring, cancelling or
invalidating a review emits no decision event, and an exact retry of a saved
decision emits no new event. Each review round has its own `reviewId`.

```json
{
  "version":"1",
  "id":"approval.4b062bc2-b366-467d-a839-8f01f7426e82",
  "event":"approval.decided",
  "timestamp":"2026-11-02T12:00:00.000Z",
  "data":{
    "accountId":"64b000000000000000000101",
    "workspaceId":"64b000000000000000000201",
    "postGroupId":"64b000000000000000000301",
    "reviewId":"4b062bc2-b366-467d-a839-8f01f7426e82",
    "decision":"approved",
    "decidedAt":"2026-11-02T12:00:00.000Z",
    "targets":[{"connectionId":"64b000000000000000000401","channelKey":"linkedin-employee","platform":"linkedin","platformId":"employee"}],
    "publication":{"scheduledTime":"2026-11-02T12:10:00.000Z","status":"scheduled"}
  }
}
```

`publication` appears only for a review requested with a `scheduledTime` and
describes the state when the decision was saved, not the current queue. An
approval is not a publication receipt: `post.published` / `post.failed` report the
outcome separately and may arrive in any order relative to the decision. The
payload contains no review link, response token, email, feedback or post content;
read the approval for feedback.

Decision delivery is durable but bounded. The decision and one delivery per
matching active subscription are saved together. The envelope, including `id` and
`timestamp`, never changes across retries. Each delivery gets up to five attempts
with delays of 1 minute, 5 minutes, 30 minutes and 2 hours, a ten-second timeout
and 2xx success; network errors and timeouts retry with the same `id`. Pending
deliveries expire after seven days. Five consecutive failures still disable the
subscription. Rotating the secret signs the next attempt with the new secret.
Changing the URL, disabling or deleting the subscription, or removing
`approval.decided` cancels its pending decision deliveries; re-enabling does not
replay them.

Verify the signature as above, then save the decision and its `id` in one
transaction before returning 2xx. A lost acknowledgement delivers the same event
again. Deduplicate decisions by envelope `id`, and keep those records longer than
the seven-day delivery window; keep publication deduplication by `event + postId`.
A late event for an older round must not replace the current round. Subscribe
before requesting reviews, and after a missed or expired delivery recover through
`GET /company/post-groups/:postGroupId/approval` (with
`Accept: application/vnd.publora.company-approval.v2+json, application/json`) and
`GET /company/post-groups/:postGroupId`. See
[Agency LinkedIn Page reshares](agency-linkedin-reshares.md) for a complete flow.

| Status/code | Meaning | Recovery |
| --- | --- | --- |
| 400 `COMPANY_WEBHOOK_INPUT_INVALID` | Invalid fields, URL, events or key | Correct the request. |
| 403 `AGENCY_CORE_ACCESS_DENIED` | Missing current Workspace settings authority | Refresh context and use an accessible Workspace. |
| 404 `COMPANY_WEBHOOK_NOT_FOUND` | No such subscription in the selected Workspace | Refresh its list. |
| 409 `COMPANY_WEBHOOK_LIMIT` | Ten subscriptions already exist | Delete an unused subscription. |
| 409 `COMPANY_WEBHOOK_IDEMPOTENCY_CONFLICT` | Key reused for a different operation | Keep the original key for its original retry; use a fresh key for a new operation. |
| 409 `COMPANY_WEBHOOK_IDEMPOTENCY_IN_FLIGHT` | Original request is still running | Respect Retry-After and retry the identical request/key. |
| 409 `COMPANY_WEBHOOK_CONFLICT` | Workspace state changed | Refresh before a new intended operation. |
| 409 `COMPANY_WEBHOOK_CLIENT_UPDATE_REQUIRED` | The request involves `approval.decided` without the v2 `Accept` header | Send `Accept: application/vnd.publora.company-webhooks.v2+json`. |
| 503 `COMPANY_WEBHOOK_UNAVAILABLE` | Dependency or transaction unavailable | Retry the identical request/key later. |

See [Agency platform settings](agency-platform-settings.md) and
[Company MCP tools](../mcp/tools-reference.md#company-tools-49).
