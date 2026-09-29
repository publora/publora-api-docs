# Agency first comment

Agency Workspace posts support one text comment under the Company's own
published post. Save it with the ordinary Company draft create or update
command:

- `POST /api/v1/company/post-groups`
- `PATCH /api/v1/company/post-groups/:postGroupId`
- MCP `company_save_post` with `draft.firstComment`

Use the existing Company authentication and exact
`X-Publora-Workspace-Id`. The draft field is
`firstComment: { "text": "Details: https://example.com", "platforms": ["linkedin"] }`.
`platforms` contains platform **types**, never Company channel keys; omit it
for every supported target. The same shape, 25,000-character absolute cap and
per-platform length rules as the [personal first comment](../endpoints/create-post.md#first-comment)
apply. `firstComment: null` clears it on update. A comment cannot be added after
publishing.

Read `GET /api/v1/company/post-groups/:postGroupId` or MCP `company_posts`
action `detail` for the stored `firstComment` and each target's
`firstCommentResult` (`pending`, `posted`, `failed`, or `skipped`). A result can
include `commentId`, `skipReason`, and `error.outcomeUnknown`. When the outcome
is unknown, inspect the live target before any manual comment. Tenant posts
are attempted only while the target connection and Workspace policy still
authorize publishing; otherwise the comment is skipped with a reason.

Agency currently accepts **one** comment. `firstComments[]` and
`delaySeconds` apply only to personal posts.
