# LinkedIn Feed Retrieval

> **Not available.** Publora does not currently expose LinkedIn feed-retrieval endpoints.

The proposed endpoints for reading LinkedIn posts, comments, and reactions are disabled. Requests to those paths currently return `404`; the route registrations are commented out in the deployed API.

This capability requires LinkedIn approval for the restricted `r_member_social` permission. Publora has not enabled it while that approval is unavailable.

## Contract status

The request parameters, response bodies, caching behavior, and MCP tools for feed retrieval are not a public contract. Do not build integrations against previously published examples or inferred endpoint shapes.

Available LinkedIn features are documented in:

- [LinkedIn platform guide](../platforms/linkedin.md)
- [LinkedIn statistics](linkedin-statistics.md)
- [LinkedIn profile summary](linkedin-profile-summary.md)
- [LinkedIn comments](linkedin-comments.md)
- [LinkedIn reactions](linkedin-reactions.md)
- [LinkedIn reshare](linkedin-reshare.md)

This URL is retained so existing links resolve to the current availability notice. Feed retrieval will receive complete contract documentation if and when the routes ship.
