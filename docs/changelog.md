# API Changelog and Deprecation Ledger

This page records externally relevant REST and MCP contract changes. Dates are deployment or documentation publication dates, not announcement estimates.

## Upcoming

### 2026-08-25 — scheduledTime strict-mode ramp

- **Affected surface:** REST `create-post` and `update-post`, plus MCP tools that schedule through them.
- **Tag:** **Potentially breaking**, configuration-dependent.
- **Change:** A `scheduledTime` at least five minutes in the past is scheduled to return `400 SCHEDULED_TIME_IN_PAST` starting on 2026-08-25. This calendar behavior applies only when production configuration does not explicitly override it with `SCHEDULED_TIME_STRICT`; an explicit flag wins in either direction.
- **Migration action:** Always send a future ISO 8601 UTC time. During the warn-first period, inspect `warnings[].code === "SCHEDULED_TIME_COERCED"` and the returned `scheduledTime` to find callers that need correction.

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
