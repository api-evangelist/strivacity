---
name: strivacity-admin-api-access-token
description: Obtain a scoped, audience-bound access token for the Strivacity Admin API using the OAuth 2.0 client-credentials flow, and diagnose the 403 that means a policy — not the token — is the blocker.
api: Strivacity Admin API
operations:
  - tokenpost
generated: '2026-08-29'
method: generated
source: https://docs.strivacity.com/reference/getting-started-with-the-admin-api
---

# Get an Admin API access token

Every Strivacity REST call is bearer-authorized. Tokens come from the tenant's own token endpoint.

## Preconditions the token request cannot fix

1. The client must be created as **"OIDC using no-code components"**. Other client types cannot be assigned an API Access policy and will not appear in the assignment dropdown.
2. Interactive login must be **disabled** on that client.
3. An **API Access policy** in the instance must already grant the client the scopes you are about to request. Requesting an ungranted scope does not error at the token endpoint in a useful way — it surfaces later as a `403` on the API call.

## Steps

1. `tokenpost` — `POST https://{tenant}.strivacity.com/oauth2/token`

   Authenticate with HTTP Basic using `client_id` as the username and `client_secret` as the password. Send `application/x-www-form-urlencoded` with:

   - `grant_type=client_credentials`
   - `audience=https://{tenant}.strivacity.com`
   - `scope=<OPTION:ENTITY ...>` — space-delimited, e.g. `read:account write:account`

   The response carries `access_token`, `expires_in` (3599s in the published example), `scope` and `token_type: bearer`.

2. Call the API with `Authorization: Bearer <access_token>` and `Accept: application/json`.

## Scope grammar

`OPTION:ENTITY`, where OPTION is `read`, `write` or `delete`. The full catalogue of 106 published scopes is in `scopes/strivacity-scopes.yml`.

## Reading failures

- **401** — the token is missing, expired, or not valid for this instance. Re-mint it.
- **403** — the token is valid and the *policy* is the blocker. Retrying will never succeed. An administrator has to grant the scope through an API Access policy. Do not treat this as a transient error and do not back off and retry.
- **429** — a rate-limit bucket is exhausted. The Administrative API family allows 50 req/s per instance and 10 req/s per source IP. Read the positionally-matched `X-Ratelimit-Limit` / `X-Ratelimit-Remaining` / `X-Ratelimit-Reset` header sets and pause for the `Reset` of whichever bucket hit zero. There is no `Retry-After`.

Errors are `application/problem+json`. Match on `errorKey`, not on `title` or `detail`.

## Caveat for code generators

The published OpenAPI documents declare only `http`/`bearer` — they do **not** declare an `oauth2` securityScheme with this token URL or these scopes. A generated client will know it needs a bearer token but not how to get one. Wire the token endpoint by hand.
