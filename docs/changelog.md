# API Changelog and Deprecation Ledger

This page records externally relevant REST and MCP contract changes. Dates are deployment or documentation publication dates, not announcement estimates.

## Upcoming

### 2026-08-25 — scheduledTime strict-mode ramp

- **Affected surface:** REST `create-post` and `update-post`, plus MCP tools that schedule through them.
- **Tag:** **Potentially breaking**, configuration-dependent.
- **Change:** A `scheduledTime` at least five minutes in the past is scheduled to return `400 SCHEDULED_TIME_IN_PAST` starting on 2026-08-25. This calendar behavior applies only when production configuration does not explicitly override it with `SCHEDULED_TIME_STRICT`; an explicit flag wins in either direction.
- **Migration action:** Always send a future ISO 8601 UTC time. During the warn-first period, inspect `warnings[].code === "SCHEDULED_TIME_COERCED"` and the returned `scheduledTime` to find callers that need correction.

## 2026-08-24

### X replies and quote posts — publora.com #405

- **Affected surface:** REST `create-post` and `update-post`, the MCP `create_post` and `update_post` tools, and the `posts[].error.code` reported by `GET /get-post` and the `post.failed` webhook.
- **Tag:** Additive. No existing request shape changes behavior.
- **Changes:**
  - `platformSettings` accepts a new top-level `twitter` object with two string keys, `replyTo` and `quoteTweet`. Both take a full `x.com`/`twitter.com` status URL or a bare 1–19 digit post ID, and both are normalized to the numeric ID before storage, so `GET /get-post` echoes the ID rather than the URL you sent.
  - `replyTo` publishes the post — or the head part of a thread — as a reply to the target; the remaining thread parts chain under it as before. `quoteTweet` applies to the single post or the thread head only. The two fields combine with each other and with media. An empty string clears either one.
  - A malformed reference is rejected at intake with `400` and a plain `error` message (`platformSettings.twitter.replyTo must be a tweet URL (https://x.com/user/status/123...) or a numeric tweet ID`); no `code` field accompanies it. An unknown key under `twitter` is still `400 PLATFORM_SETTING_UNKNOWN`, which is evaluated first.
  - Two publish-time codes were added: `X_REPLY_NOT_AUTHORIZED` when X refuses the reply or quote relationship, and `X_TARGET_REJECTED` when the target is deleted, protected, or its author blocked the account. Both are permanent — `retryable: false`.
  - `twitter` is no longer an example of a rejected `platformSettings` platform; the allowlist now has seven platforms.
- **Restriction to know before integrating:** X allows a programmatic reply or quote on self-serve API tiers only when the target post's author mentioned the connected account **in that same post**, quoted one of the account's posts, or the connected account authored the target. Enterprise apps are exempt. Publora cannot check that relationship at intake, so an unrelated target is accepted by `create-post`/`update-post` and fails later at publish time.
- **Migration action:** None for existing callers. New integrations should treat these fields as inbound-engagement and own-post tools, match on `error.code` rather than message text, and not retry `X_REPLY_NOT_AUTHORIZED` or `X_TARGET_REJECTED` with the same target.

## 2026-08-03

### MCP OAuth consent no longer asks for an API key

- **Affected surface:** the OAuth 2.1 flow on `mcp.publora.com` (Dynamic Client Registration + PKCE), used by claude.ai's custom connector, Claude Code, Codex/ChatGPT, Cursor, VS Code, Manus and other browser-capable clients.
- **Tag:** Behavioral, non-breaking for existing credentials.
- **Change:** The consent step is a sign-in-and-approve page — *"An application is requesting access to your Publora account"*, naming the account being authorized, with **Approve** and **Cancel**. It no longer asks you to paste an `sk_...` key; that step was replaced when SSO login shipped on 2026-07-27, and the wording was finalized on 2026-08-03. On approval Publora mints a dedicated API key for that client registration, named `MCP (<client> #<id>)`, and returns it as the access token. Static API-key headers (`Authorization: Bearer sk_...` / `x-publora-key`) are unaffected and remain the option for headless clients.
- **Migration action:** None for connectors that already work. Re-authorizing a client mints a fresh key; per-client keys are listed and revocable on the **API** page in the dashboard. If you built on the old instructions and expected a key-paste page, drop that step.

## 2026-07-21

### Editable draft and scheduled posts — publora.com #231

- **Affected surface:** REST `PUT /update-post/:postGroupId` and the MCP `update_post` tool.
- **Tag:** Additive, with new stable conflict codes.
- **Changes:**
  - `update-post` accepts two new optional patch fields: `content` (replacement base text) and `platforms` (replacement target set). Omitting a field leaves the stored value unchanged.
  - A `content` edit rewrites every platform post to its effective text while preserving explicit per-account overrides; editing the text of a Twitter or Threads target clears its derived thread split.
  - A `platforms` edit replaces the whole target set: dropped IDs have their platform posts deleted, added IDs are validated for ownership and plan entitlement, and adding a target to a scheduled post re-runs scheduling limits plus full content/media validation. `[]` is accepted only while the post remains a draft.
  - Added stable codes `POST_NOT_EDITABLE` (400), `POST_PUBLISH_IN_PROGRESS` (409), and `POST_GROUP_VERSION_CONFLICT` (409), plus `INVALID_CONTENT`, `INVALID_PLATFORMS`, `INVALID_PLATFORM_CONNECTION`, and `INVALID_PLATFORM_ID` (400). The pre-existing `"Cannot update post: post is currently in {status} status"` 400 now also carries `code: "POST_NOT_EDITABLE"`.
  - The success snapshot's `postGroup` now always includes the effective `content` and `platforms`.
  - The "at least one field" error text changed to `"At least one of status, scheduledTime, content, platforms, platformSettings, or mediaUrls must be provided"`.
  - `platforms` arrays on both `create-post` and `update-post` now reject duplicate connection IDs with `400 "Platforms must not contain duplicates"`.
  - MCP `update_post` exposes `content` and `platforms`, and instructs clients to call `list_connections` before changing targets.
- **Migration action:** No change is required for existing callers. Integrations that previously deleted and recreated a post to fix its text or targets should switch to `update-post`. Match on the new `code` values rather than message text, send an `Idempotency-Key` with content/platform edits, and re-read the post with `GET /get-post` on a 409 instead of blind-retrying. If you relied on a repeated connection ID in `platforms` being tolerated, de-duplicate the array.

## 2026-07-15

### API/MCP correctness release — publora.com #198

- **Affected surface:** REST create/update/get-post, webhooks, MCP sessions and tool responses.
- **Tags:** **Breaking** for unknown `platformSettings` paths; additive/behavioral for the remaining items.
- **Changes:**
  - Added Mongo-backed `Idempotency-Key` handling to create/update.
  - Added the warn-first `scheduledTime` ramp and structured coercion/rejection codes.
  - Unknown `platformSettings` paths now return `400 PLATFORM_SETTING_UNKNOWN`; they are no longer silently discarded.
  - Added MCP session admission checks and more robust MCP response parsing.
  - Publication identity now flows through get-post and `post.published`, including `platformId`, `postedId`, and nullable `permalink`.
- **Migration action:** Remove unknown settings before retrying; add idempotency keys to retryable create/update workflows; read publication identity from documented fields; audit past-time warnings before strict mode.

### LinkedIn repost/reshare — publora.com #194

- **Affected surface:** REST, MCP, and scheduled LinkedIn publishing.
- **Tag:** Additive.
- **Changes:** Added `POST /linkedin-reshare`, the `linkedin_create_reshare` MCP tool (the 14th active tool), and `platformSettings.linkedin` with `repostEnabled`, `repostParentUrn`, and `repostVisibility` as the sixth accepted settings branch.
- **Migration action:** No change is required for existing callers. New integrations should use a `urn:li:share:*` or `urn:li:ugcPost:*` parent. `CONNECTIONS` visibility is personal-profile-only; use `PUBLIC` for company-page reposts.

### Correctness-release documentation — publora-api-docs #21

- **Affected surface:** Public API, MCP, webhook, and OpenAPI documentation.
- **Tag:** Documentation.
- **Change:** Published the idempotency, scheduled-time, strict-settings, MCP-draft, and publication-identity contract introduced by the correctness release.
- **Migration action:** Compare existing integrations with the updated create/update, get-post, webhook, and MCP reference pages; no separate runtime change was introduced by the docs release.

## Deprecation policy

No additional public API deprecations are currently scheduled. Future entries will identify the affected surface, breaking behavior, effective date, and migration action.
