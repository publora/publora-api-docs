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
recommended. Subscribe to one or both of `post.published` and `post.failed`;
other events and duplicate event names are rejected. `isActive` defaults to true.
The limit is ten subscriptions per Workspace, including disabled subscriptions.

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

Events describe an individual platform post, not completion of the entire
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

Delivery reuses the existing best-effort transport: one HTTP attempt per dispatch,
a ten-second timeout, 2xx success only and no automatic HTTP retry queue. Duplicate
dispatches are possible; process receivers idempotently using `event + postId`
(and your stored processing state), not `postGroupId` alone. There is no durable
outbox or exactly-once guarantee. After five consecutive failures, a subscription
is disabled. Re-enable it after correcting its URL/receiver; enabling resets the
failure count. A delivery already started may finish after disable, deletion or
secret rotation.

| Status/code | Meaning | Recovery |
| --- | --- | --- |
| 400 `COMPANY_WEBHOOK_INPUT_INVALID` | Invalid fields, URL, events or key | Correct the request. |
| 403 `AGENCY_CORE_ACCESS_DENIED` | Missing current Workspace settings authority | Refresh context and use an accessible Workspace. |
| 404 `COMPANY_WEBHOOK_NOT_FOUND` | No such subscription in the selected Workspace | Refresh its list. |
| 409 `COMPANY_WEBHOOK_LIMIT` | Ten subscriptions already exist | Delete an unused subscription. |
| 409 `COMPANY_WEBHOOK_IDEMPOTENCY_CONFLICT` | Key reused for a different operation | Keep the original key for its original retry; use a fresh key for a new operation. |
| 409 `COMPANY_WEBHOOK_IDEMPOTENCY_IN_FLIGHT` | Original request is still running | Respect Retry-After and retry the identical request/key. |
| 409 `COMPANY_WEBHOOK_CONFLICT` | Workspace state changed | Refresh before a new intended operation. |
| 503 `COMPANY_WEBHOOK_UNAVAILABLE` | Dependency or transaction unavailable | Retry the identical request/key later. |

See [Agency platform settings](agency-platform-settings.md) and
[Company MCP tools](../mcp/tools-reference.md#agency-tools-13).
