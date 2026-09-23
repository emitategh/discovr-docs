# Discovr / Sonas — MCP Reviewer Guide

Companion to our [Anthropic Connector Directory](https://claude.com/docs/connectors/building/submission) submission. Walks a reviewer through authenticating, exercising every tool, and verifying expected behaviour. The same backend serves both brand fronts — listings are split by brand only so each surfaces under its real name in the catalogue.

## At a glance

| Brand   | MCP endpoint                       | OAuth metadata                                                    | Web                  |
|---------|------------------------------------|-------------------------------------------------------------------|----------------------|
| Discovr | `https://mcp.discovr.es/mcp/`      | `https://mcp.discovr.es/.well-known/oauth-protected-resource`     | `https://discovr.es` |
| Sonas   | `https://mcp.sonas.work/mcp/`      | `https://mcp.sonas.work/.well-known/oauth-protected-resource`     | `https://sonas.work` |

Transport: **Streamable HTTP**. Both endpoints return `401 + WWW-Authenticate: Bearer resource_metadata=...` for unauthenticated requests, so MCP clients auto-discover the OAuth flow without manual configuration.

## Test accounts

We provide one test account per brand in the submission form's "Test credentials" section. Each account already has:

- A populated profile (CV uploaded, interview answers in, profile published) so the recruiter-side tools have something real to read.
- A linked recruiter role on the same email, so a single sign-in lets the reviewer exercise both the candidate and recruiter tool sets without account switching.
- A non-zero token balance so `chat` succeeds without paywall friction.

If credentials need to be rotated mid-review, email `privacy@discovr.es` and we'll regenerate within one business day.

## Authentication flow

1. Add the connector via the Claude / ChatGPT catalogue, or paste the MCP URL directly.
2. The client opens an OAuth window pointing at `https://mcp.discovr.es/authorize` (or `https://mcp.sonas.work/authorize`), which forwards to the brand's sign-in page at `/login`.
3. Sign in with **email + password** (use the test credentials from the submission) or with Google. Both are backed by Firebase Auth.
4. After consent, control returns to the MCP client with an opaque token.
5. The client makes its first MCP call. The token is validated against our DB and a `User` row is attached to the request context for the lifetime of that call.

A successful first call should be the `help` tool — it has no role requirement and no side effects, so it confirms transport + token plumbing in one step.

## Tool inventory

All tools surface a `title` plus `readOnlyHint` / `destructiveHint` annotations. The matrix:

| Tool             | Audience   | Title                       | Read-only | Destructive | Notes                                                       |
|------------------|------------|-----------------------------|-----------|-------------|-------------------------------------------------------------|
| `help`           | any        | Help                        | ✅        | ❌          | Static text. Any signed-in user, no role required.          |
| `get_candidate`  | recruiter  | Get candidate               | ✅        | ❌          | Returns 404 if `published_at` is null or username unknown.  |
| `chat`           | recruiter  | Chat with candidate AI      | ❌        | ❌          | Persists messages + bills the recruiter; not destructive.   |
| `get_my_profile` | candidate  | View my profile             | ✅        | ❌          | Includes completeness score and missing-fields list.        |
| `create_profile` | candidate  | Create or replace profile   | ❌        | ✅          | "Create OR REPLACE" — overwrites any existing draft.        |
| `update_profile` | candidate  | Update profile field        | ❌        | ✅          | Overwrites the named field. Resets `published_at` to null.  |
| `publish`        | candidate  | Publish profile             | ❌        | ❌          | Flips a flag and re-runs bio generation. Nothing is lost.   |

Role enforcement is server-side: a candidate calling `chat` gets back an `McpError` (HTTP 403 in the result, no data leak).

## Suggested review pass

A reviewer can cover the full surface in ~10 minutes:

1. **`help`** — verify transport + auth wiring. Expected: JSON with `description`, `auth`, `tools`, `resources`, `quick_start`.
2. **`get_my_profile`** — confirms candidate auth and that the test account has a real profile. Expected: `completeness: 100`, no items in `missing`.
3. **`update_profile`** with `{"field": "title", "value": "Senior Backend Engineer"}` — confirms write path + the `published_at` reset. Re-call `get_my_profile` to see `status: "draft"` afterward.
4. **`publish`** — re-publish the profile so the recruiter tools have published data to read.
5. **`get_candidate`** with the test username — confirms recruiter read path. Expected: name, title, skills, experience, languages, work_authorization.
6. **`chat`** with `{"username": "<test-username>", "message": "What's your strongest project?"}` — confirms LLM path. Reply should be in first person and reference content from the candidate's interview answers. Output language follows the recruiter's locale (set via Discovr's in-app language toggle; default English).
7. **`create_profile`** — only run if you want to exercise the destructive-replace path. The test account profile will be reset to the new shape; we'll restore it after review.

## Known limits

- **Free-tier caps**: recruiters get 5 candidates / 3 messages each. The reviewer account is provisioned past those caps so the cap UI doesn't fire during review.
- **Chat latency**: 3-8 seconds per turn (Claude Sonnet). The reply is returned in a single tool result once generation finishes.
- **Embeddings**: profile text is embedded via OpenAI `text-embedding-3-small`. New profiles take 5-15 seconds before they're searchable.
- **Data residency**: app DB on Hostinger (EU). Anthropic + OpenAI + Stripe + Sentry are US sub-processors under EU SCCs. Detail at the [privacy policy](https://discovr.es/privacy).

## Escalation

| Issue                                       | Contact                            |
|---------------------------------------------|-------------------------------------|
| Test credentials revoked / not working      | `privacy@discovr.es`                |
| Tool returns unexpected error               | `privacy@discovr.es` with the JSON  |
| Submission status / re-review request       | `privacy@discovr.es`                |

## See also

- Public MCP Registry listings: [`es.discovr/profiles`](https://registry.modelcontextprotocol.io/v0/servers?search=es.discovr/profiles) · [`work.sonas/profiles`](https://registry.modelcontextprotocol.io/v0/servers?search=work.sonas/profiles)
- Privacy policy: https://discovr.es/privacy · https://sonas.work/privacy
- OAuth metadata: https://mcp.discovr.es/.well-known/oauth-protected-resource
