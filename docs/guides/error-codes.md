# API Error Code Catalog

This catalog covers stable codes reachable through API-key-authenticated `/api/v1` routes. It excludes internal X Bookmarks codes and codes emitted only by dashboard sessions, OAuth, billing, or other non-API-key routes.

Codes can appear at the top level, inside `warnings[]`, inside `validation.errors[]`, per URL in `mediaResults[]`, or per connection in `issues` on the analytics routes. Match on the code, not the human-readable message.

For retry patterns and idempotency guidance, see [Error Handling](./error-handling.md).

## Request and scheduling codes

| Code | HTTP | Location | Meaning | Retryable? | Recovery |
|---|---:|---|---|---|---|
| `IDEMPOTENCY_IN_FLIGHT` | 409 | top-level `code` | Another request with this idempotency key is still running, or its claim was superseded | Yes, shortly | Retry the identical request with the same key |
| `IDEMPOTENCY_KEY_CONFLICT` | 422 | top-level `code` | The key was already used with a different body or resource | No, unchanged | Replay the original request or use a new key for the new operation |
| `IDEMPOTENCY_BODY_TOO_COMPLEX` | 400 | top-level `code` | The body is too deeply nested to hash safely for idempotency | No | Simplify the request body or omit `Idempotency-Key` if safe retries are not required |
| `PLATFORM_SETTING_UNKNOWN` | 400 | top-level `code` | A `platformSettings` path is outside the strict allowlist | No | Remove or correct the path named by `field` |
| `PLATFORMS_REQUIRED` | 400 | top-level `code` | Scheduling would leave the post with no target platforms | No | Add at least one platform, or keep the post as a draft |
| `INVALID_CONTENT` | 400 | top-level `code` | `update-post` received a `content` value that is not a string | No | Send `content` as a string; `field` names the offending key |
| `INVALID_PLATFORMS` | 400 | top-level `code` | `update-post` received a `platforms` value that is not a valid, duplicate-free array of connection IDs | No | Send a de-duplicated array of IDs from `GET /platform-connections`; `field` names the offending key |
| `INVALID_PLATFORM_CONNECTION` | 400 | top-level `code` | A platform ID added by an `update-post` edit is not a connection owned by the acting user | No | Use IDs from `GET /platform-connections`; the offending IDs are listed in `invalidPlatforms` |
| `INVALID_PLATFORM_ID` | 400 | top-level `code` | A stored or supplied platform key could not be parsed while reconciling platform posts | No | Send IDs in the `<platform>-<platformId>` form returned by `GET /platform-connections` |
| `POST_NOT_EDITABLE` | 400 | top-level `code` | The post group, or one of its platform posts, is past the editable `draft`/`scheduled` window | No | Create a new post; published and failed posts cannot be edited |
| `POST_PUBLISH_IN_PROGRESS` | 409 | top-level `code` | Publishing has already started for the group or one of its platform posts | Only after re-checking | Re-read with `GET /get-post`; retry only while the post is still `draft` or `scheduled` |
| `POST_GROUP_VERSION_CONFLICT` | 409 | top-level `code` | Another write committed first; this request changed nothing | Yes, after re-reading | Re-read with `GET /get-post`, rebase the edit on the current state, and retry |
| `SCHEDULED_TIME_IN_PAST` | 400 | top-level `code` | Strict mode rejected a time at least five minutes in the past | No | Send a future ISO 8601 UTC time |
| `SCHEDULED_TIME_COERCED` | 200 | `warnings[].code` | A past time was clamped to server time during the warn-first ramp | Not an error | Read the returned `scheduledTime` and fix the caller to send a future time |
| `MEDIA_VALIDATION_PENDING` | 200 | `warnings[].code` | Scheduling succeeded while bounded media probing remained transiently incomplete | Not an error | Monitor the post; media is checked again before publishing |
| `MEDIA_NOT_READY` | 400 | top-level `code` | Attached upload bytes are missing, still uploading, or previously failed validation | After fixing media | Complete/re-upload the media, then schedule again |
| `MEDIA_URL_RATE_LIMITED` | 429 | top-level `code` | The fixed-window allowance of 60 ingested URLs was exceeded | Yes | Wait the `Retry-After` seconds, then retry; preserve the idempotency key when applicable |

## Plan and workspace limit codes

These top-level codes are `LimitExceededError` responses with HTTP 403.

| Code | Meaning | Retryable? | Recovery |
|---|---|---|---|
| `CONNECTIONS_OVER_LIMIT` | The workspace already has more connected channels than its plan permits | After account changes | Disconnect channels or upgrade before scheduling |
| `CHANNEL_LIMIT_REACHED` | Adding or attaching would exceed the channel allowance | After account changes | Disconnect a channel or upgrade |
| `SCHEDULE_HORIZON_REACHED` | The requested time is farther ahead than the plan permits | No, unchanged | Choose an earlier time or upgrade |
| `SCHEDULED_POST_LIMIT_REACHED` | The active scheduled-post queue is full | After capacity frees | Wait for or delete an existing scheduled post, or upgrade |
| `POST_LIMIT_REACHED` | The monthly account/connection post allowance is exhausted | After reset or upgrade | Wait for `periodEnd` or upgrade |
| `PLATFORM_NOT_AVAILABLE` | The plan does not allow one or more selected platforms | No, unchanged | Remove the platforms listed in the top-level `disallowedPlatforms` array or upgrade |

## Media completion codes

These codes are returned by `POST /complete-media/{mediaFileId}`.

| Code | HTTP | Location | Meaning | Retryable? | Recovery |
|---|---:|---|---|---|---|
| `INVALID_MEDIA_FILE_ID` | 400 | top-level `code` | `mediaFileId` is not a Mongo ObjectId | No | Use the `mediaId` returned by `get-upload-url` |
| `MEDIA_FILE_NOT_FOUND` | 404 | top-level `code` | No media row belongs to the caller for that ID | No | Check the ID and API-key owner; upload again if it was deleted |
| `MEDIA_FILE_INDETERMINATE_TYPE` | 400 | top-level `code` | Neither stored type nor MIME prefix identifies image, video, or PDF | No | Re-upload with the correct `Content-Type` |
| `PROBE_FAILED` | 400 | top-level `code` | The stored media already failed byte-level validation | No | Re-upload a valid file |
| `MEDIA_FILE_MISSING_URL` | 500 | top-level `code` | The media row has no storage URL | No client-side retry contract | Upload again; contact support if newly created rows repeat this state |
| `PROBE_VERSION_UNVERIFIED` | 503 | top-level `code` | The object version could not be verified before or after probing | Yes | Retry `complete-media` after a few seconds |
| `PROBE_DOWNLOAD_FAILED` | 503 | top-level `code` | A transient S3/CDN download prevented probing | Yes | Retry `complete-media` after a few seconds |
| `PROBE_VERSION_CHANGED` | 503 | top-level `code` | The uploaded object changed while it was being validated | Yes, after upload stabilizes | Stop overwriting the presigned object and retry |
| `MEDIA_FILE_RACE` | 409 | top-level `code` | The row was deleted or failed concurrently during probing | No | Re-upload and use the new media ID |

## `mediaUrls` per-URL codes

When create/update ingestion fails, the request returns HTTP 400 with `error: "Media ingestion failed"`; the URL/fetch codes below can appear in `mediaResults[].code`. Probe failures are forwarded into the same field using the applicable code from [Media completion codes](#media-completion-codes). The batch is all-or-nothing.

| Code | HTTP | Location | Meaning | Retryable? | Recovery |
|---|---:|---|---|---|---|
| `MEDIA_URL_INVALID` | 400 | `mediaResults[].code` | The value is not a URL | No | Supply a valid public URL |
| `MEDIA_URL_PROTOCOL` | 400 | `mediaResults[].code` | The URL is not HTTPS | No | Use `https://` |
| `MEDIA_URL_CREDENTIALS` | 400 | `mediaResults[].code` | The URL embeds a username or password | No | Use a credential-free URL, such as a signed HTTPS URL |
| `MEDIA_URL_PORT` | 400 | `mediaResults[].code` | The URL uses a non-default HTTPS port | No | Serve it on port 443 |
| `MEDIA_URL_BLOCKED_HOST` | 400 | `mediaResults[].code` | The host or resolved address is private/blocked | No | Use a publicly reachable host |
| `MEDIA_URL_DNS` | 400 | `mediaResults[].code` | DNS returned no usable address | After DNS is fixed | Correct DNS, then retry the whole request |
| `MEDIA_URL_REDIRECT` | 400 | `mediaResults[].code` | A redirect lacked a `Location` header | After the origin is fixed | Use the final URL or fix the redirect |
| `MEDIA_URL_TOO_MANY_REDIRECTS` | 400 | `mediaResults[].code` | The fetch exceeded the redirect limit | No, unchanged | Use a direct URL |
| `MEDIA_URL_HTTP_ERROR` | 400 | `mediaResults[].code` | The remote server returned a non-200 response | After the origin is available | Check access/expiry and retry with a working URL |
| `MEDIA_URL_INGEST_TIMEOUT` | 400 | `mediaResults[].code` | The request-wide media ingestion budget expired | Conditionally | Use a faster origin, smaller media, or the presigned upload flow |
| `MEDIA_URL_TOO_LARGE` | 400 | `mediaResults[].code` | One image/video exceeds the URL-ingestion byte cap | No | Use a smaller file or `get-upload-url` |
| `MEDIA_URL_AGGREGATE_TOO_LARGE` | 400 | `mediaResults[].code` | Combined downloaded bytes exceed 300 MiB | No | Reduce the batch or use presigned uploads |
| `MEDIA_URL_UNSUPPORTED_FORMAT` | 400 | `mediaResults[].code` | Magic-byte detection found an unsupported format | No | Convert to a supported image/video format |
| `MEDIA_URL_FETCH_FAILED` | 400 | `mediaResults[].code` | An unexpected network/download failure occurred | Conditionally | Verify the URL and retry once; otherwise use presigned upload |
| `MEDIA_URL_PROBE_FAILED` | 400 | `mediaResults[].code` | Defensive fallback if a future failed probe supplies no specific code; current `probeAndPersistMedia` failure branches always supply one | Not currently expected | If observed, convert or re-upload valid media and report the unexpected fallback |
| `MEDIA_URL_SKIPPED` | 400 | `mediaResults[].code` | This URL was not attempted after another URL failed | After fixing the root failure | Correct the failing URL and retry the complete batch |
| `MEDIA_URL_ROLLED_BACK` | 400 | `mediaResults[].code` | This URL succeeded, but its media was removed because another URL failed | After fixing the root failure | Correct the failing URL and retry the complete batch |

## Mastodon and Bluesky analytics codes

`POST /post-statistics` reports per-connection problems in the `issues` object of a `200` response, keyed by connection ID (`<platform>-<platformId>`); the affected `postedId` values are `null` in `stats`. `POST /profile-statistics` reports the same conditions in `unavailable` (`AUTH_REVOKED`, `FORBIDDEN`) and `rateLimited`. See [Mastodon and Bluesky Statistics](../endpoints/platform-statistics.md).

| Code | HTTP | Location | Meaning | Retryable? | Recovery |
|---|---:|---|---|---|---|
| `ANALYTICS_PLAN_REQUIRED` | 403 | top-level `code` | The plan has no analytics feature | No, unchanged | Upgrade to a plan that includes analytics |
| `ANALYTICS_REQUEST_TIMEOUT` | 504 | top-level `code` | The request exceeded its 90-second budget | Yes | Retry with fewer posts per request |
| `CONNECTION_NOT_FOUND` | 200 | `issues[<connection>]` | No connection of the caller matches that platform and platform ID | No, unchanged | Re-read `GET /platform-connections` |
| `AUTH_REVOKED` | 200 | `issues[<connection>]`, `unavailable` | Mastodon rejected the stored token (HTTP 401) | After reconnecting | Reconnect the account |
| `FORBIDDEN` | 200 | `issues[<connection>]`, `unavailable` | Mastodon refused analytics access (HTTP 403, scope or moderation). Publishing is unaffected. | Later | Retry later; reconnect if it persists |
| `RATE_LIMITED` | 200 | `issues[<connection>]` | The platform is in a cooldown after answering 429 | Yes, after the cooldown | Retry later; the response also carries `rateLimited: true` |
| `FETCH_FAILED` | 200 | `issues[<connection>]` | A transient failure reaching the platform | Yes | Retry later |

## LinkedIn symbolic errors

These LinkedIn endpoints use the `error` field for stable symbolic values rather than a separate `code` field.

| Value | HTTP | Meaning | Retryable? | Recovery |
|---|---:|---|---|---|
| `REACTION_ALREADY_EXISTS` | 409 | The actor already has a reaction on the target post | No, unchanged | Delete the existing reaction, then apply the requested type |
| `LINKEDIN_TOKEN_EXPIRED` | 401 | The stored LinkedIn access token is expired | No | Reconnect the LinkedIn account |
| `LINKEDIN_RATE_LIMITED` | 429 | LinkedIn is rate-limiting the analytics resource/account | Yes | Wait the returned `retryAfter` seconds |
| `LINKEDIN_REQUEST_TIMEOUT` | 504 | The backend request deadline elapsed while waiting for LinkedIn | Yes | Retry the read/interaction with normal backoff |

## Post validation codes

Scheduling validation returns HTTP 400. These codes appear in `validation.errors[].code`. A successful request returns no `validation` object at all, so validation warnings are not part of the success contract.

| Code | Meaning | Recovery |
|---|---|---|
| `CONTENT_TOO_LONG` | Content exceeds the selected platform's character limit | Shorten it or change the target platform/account tier |
| `CONTENT_OR_MEDIA_REQUIRED` | The post has neither usable text nor media | Add content or media |
| `MEDIA_REQUIRED` | A selected platform requires media | Attach supported images or video |
| `MEDIA_SIZE_EXCEEDED` | A file exceeds the platform byte limit | Compress or replace it |
| `MEDIA_COUNT_EXCEEDED` | Too many media items are attached | Delete extras and schedule again |
| `MEDIA_TYPE_NOT_SUPPORTED` | The media type is unsupported for the platform | Convert or remove it |
| `MEDIA_DIMENSIONS_INVALID` | Image/video dimensions violate a platform rule | Resize the media |
| `MEDIA_ASPECT_RATIO_INVALID` | Aspect ratio violates a platform rule | Crop or resize the media |
| `VIDEO_REQUIRED` | The selected platform requires video | Attach a supported video |
| `VIDEO_DURATION_EXCEEDED` | Video is longer than the platform maximum | Trim it |
| `VIDEO_DURATION_TOO_SHORT` | Video is shorter than the platform minimum | Use a longer video |
| `IMAGES_NOT_SUPPORTED` | The selected platform does not accept images for this post type | Remove images or change the target |
| `DOCUMENTS_NOT_SUPPORTED` | The selected platform does not accept document media | Remove the document or target LinkedIn |
| `PLATFORM_SETTING_REQUIRED` | A required platform-specific setting is absent | Supply the field identified in the error |
| `PLATFORM_SETTING_INVALID` | A platform-specific setting has an invalid value | Use one of the reported accepted values |
| `PLATFORM_NOT_SUPPORTED` | The validator has no contract for the target platform | Remove the unsupported target |

All validation codes above are non-retryable with the unchanged request.

## Threads chain codes

These codes concern multi-part Threads chains only. A single Threads post never
triggers the permission check.

### At scheduling

| Code | HTTP | Location | Meaning | Retryable? | Recovery |
|---|---:|---|---|---|---|
| `THREADS_PERMISSION_REQUIRED` | 400 | `validation.errors[].code` | The connection's token does not grant `threads_manage_replies`, or the connection no longer exists. Nothing was published | No, unchanged | Reconnect that Threads account in Publora **Channels**, approve all requested permissions, then schedule again |
| `TOKEN_EXPIRED` | 400 | `validation.errors[].code` | The stored Threads token is absent, or Meta reports it invalid | No, unchanged | Reconnect the Threads account |
| `THREADS_PERMISSIONS_UNAVAILABLE` | 503 | top-level `code` | Publora could not complete the permission check itself. This is **not** an account problem and never means reconnect | Usually | Retry scheduling in a few seconds. A lookup that keeps failing is a Publora-side configuration problem no retry will clear — contact support |
| `THREAD_PART_TOO_LONG` | 400 | `validation.errors[].code` | A resolved thread part exceeds 500 characters. Automatic splitting never produces one, but a `[n/m]` part you wrote yourself is not re-split, so an oversized one lands here | No, unchanged | Shorten that part; `field` names its index |
| `INVALID_PLATFORM_CONTENT` | 400 | `validation.errors[].code` | A stored thread part is not a text string | No, unchanged | Edit or remove the invalid part |

> **`THREADS_PERMISSIONS_UNAVAILABLE` has its own envelope.** Unlike the validation
> codes, it returns `{ "code": "THREADS_PERMISSIONS_UNAVAILABLE", "error": "…" }`
> with **no** `validation` object.

> **`THREADS_PERMISSION_REQUIRED` carries one of two messages.** A connection whose
> token lacks the grant returns *"Reconnect your Threads account and allow reply
> publishing to publish a thread with multiple posts."*; a target whose connection no
> longer exists returns *"Reconnect your Threads account before scheduling a thread."*
> Match on the code, not the message.

> **There is no way to pre-check the grant.** `GET /platform-connections` does not
> report granted scopes, so a missing permission first surfaces as
> `THREADS_PERMISSION_REQUIRED` when you schedule a chain.

### At publishing

These appear in `posts[].error.code` from `GET /get-post`, never as an HTTP status.

| Code | Meaning | Retryable? | Recovery |
|---|---|---|---|
| `THREADS_INVALID_THREAD_PART` | A part was empty, not text, or over 500 characters including its numbering | No | Fix the content and create a new post |
| `THREAD_PARTIALLY_PUBLISHED` | Some parts published, then one failed. The published parts are live and Publora will not republish the chain | No | Read `threadParts`, then publish only the missing parts |
| `PUBLISH_OUTCOME_UNKNOWN` | Threads accepted a publish request without returning an ID, so the outcome is genuinely unknown. Automatic retry is disabled to avoid duplicates | No | Check the live Threads account before posting anything further |
| `THREADS_PERMISSION_REQUIRED` | The grant disappeared between scheduling and publishing. Nothing was published | No | Reconnect the account and schedule again |

> **Never resubmit a chain after `THREAD_PARTIALLY_PUBLISHED` or
> `PUBLISH_OUTCOME_UNKNOWN`.** Parts marked `published` in `threadParts` already
> exist on the platform; recreating the post duplicates them.

## Workspace attachment codes

These lowercase codes are top-level response fields from `POST /workspace/users` or `POST /workspace/users/attach`.

| Code | HTTP | Meaning | Recovery |
|---|---:|---|---|
| `email_exists` | 409 | A user with the requested email already exists | Inspect the returned attachment status; use the attach flow when allowed |
| `missing_identifier` | 400 | Neither email nor user ID was supplied | Send one identifier |
| `invalid_user_id` | 400 | The user ID is malformed | Send a Mongo ObjectId |
| `invalid_email` | 400 | The email format is invalid | Correct the email |
| `user_not_found` | 404 | No active target user was found | Check the identifier |
| `identifier_mismatch` | 400 | Supplied email and user ID identify different users | Correct one identifier |
| `user_inactive` | 409 | The target user is inactive | Reactivate the user before attaching |
| `cannot_attach_self` | 400 | The workspace owner was supplied as the target | Choose another user |
| `user_is_workspace_owner` | 409 | The target owns a workspace | A workspace owner cannot be attached as managed |
| `user_in_another_workspace` | 409 | The target already belongs to another workspace | Detach it from the other workspace first |
| `user_has_active_subscription` | 409 | The target has a blocking subscription | Resolve that subscription before attaching |
| `attach_consent_required` | 403 | Attaching an existing standalone user requires its consent token | Supply the target user's access token |
| `invalid_attach_consent` | 403 | The consent token is invalid or revoked | Obtain a fresh token from the target user |
| `attach_consent_mismatch` | 403 | The token belongs to another user | Supply the target user's own token |

## Source coverage

- Top-level scheduling and idempotency responses: `backend/src/app/routes/apiRoute.js`.
- Plan/workspace entitlement responses: `backend/src/app/services/limitsService.js`.
- Scheduling guards and media state: `backend/src/app/helpers/scheduledPlatformsGuard.js`, `mediaStatusTransitions.js`, and `probeMediaUpload.js`.
- Per-URL ingestion: `backend/src/app/helpers/mediaUrlIngest.js` and `mediaUrlSecurity.js`.
- Validation codes actually emitted by scheduling: `backend/src/app/services/postValidationService.js` and `errors/ValidationError.js`.
- Threads chain permission and publish errors: `backend/src/app/helpers/threadsPermissions.js`, `backend/src/app/services/threadsPostPreflight.js`, and `scheduler-service/src/app/controllers/threadsController.js`.
- Workspace attachment responses: `backend/src/app/controllers/workspaceApiController.js`.
- LinkedIn interaction/analytics errors: `backend/src/app/controllers/linkedinController.js`.
- Mastodon/Bluesky analytics responses: `backend/src/app/controllers/platformAnalyticsController.js` and `backend/src/app/services/platformAnalytics/`.
