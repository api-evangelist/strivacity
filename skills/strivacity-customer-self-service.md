---
name: strivacity-customer-self-service
description: Act on a signed-in customer's own account through the Strivacity MyAccount API — read the profile, manage MFA authenticators, change the password, manage linked identities and notification preferences.
api: Strivacity MyAccount API
operations:
  - getPrincipal
  - getPersonal
  - updatePersonal
  - getAvailableAttributes
  - updateIdentifierRequest
  - updatePassword
  - addAuthenticatorRequest
  - updateAuthenticator
  - removeAuthenticatorRequest
  - getMyIdentities
  - deleteExternalIdentity
  - getNotificationPreferences
  - getNotificationPreferencesDescriptor
  - updateNotificationPreferences
generated: '2026-08-29'
method: generated
source: openapi/strivacity-myaccount-portal-openapi.yml
---

# Customer self-service

The MyAccount API acts **as the signed-in customer**, on that customer's own account. It is not an administrative surface — there is no `accountId` in any path. Base: `https://{tenant}.strivacity.com/myaccount/api/v1`.

## Steps

1. `getPrincipal` — `GET /me`. Establishes who the token belongs to. Always start here.
2. `getPersonal` / `updatePersonal` — `GET|PUT /personal` for account metadata; `getAvailableAttributes` (`GET /attributes`) lists what this identity store allows.
3. **MFA:**
   - `addAuthenticatorRequest` — `POST /authenticators` to enroll
   - `updateAuthenticator` — `POST /authenticators/{authenticatorId}` to change the methods on an enrolled authenticator
   - `removeAuthenticatorRequest` — `DELETE /authenticators/{authenticatorId}`
4. `updatePassword` — `POST /password`.
5. `updateIdentifierRequest` — `POST /identifier` to change the login identifier (email / username / phone).
6. **Linked identities:** `getMyIdentities` (`GET /identities`), `deleteExternalIdentity` (`DELETE /identities/{externalLoginId}`) to unlink a social or enterprise login.
7. **Notification preferences:** read the shape from `getNotificationPreferencesDescriptor` before calling `updateNotificationPreferences` — the descriptor is what tells you which preference keys this instance actually has.

## The X-Challenge header

Several MyAccount operations declare an `X-Challenge` request header. It carries a step-up challenge: the instance's Adaptive Access policy can require re-verification before a sensitive self-service change lands. Do not strip it, and do not treat a challenge response as a hard failure — it is a request for a second factor.

## Rate limits

Self-service paths (`/myaccount/...`) are among the tightest published buckets: **5 req/s and 50 req/min per source IP**, against 300 req/s per instance. Three MyAccount operations declare `429` directly in the spec. Serialize work on behalf of one customer; do not parallelize.

## Errors

`application/problem+json` with the `ProblemJson` envelope. `409` appears on three operations — usually a state conflict such as removing the last remaining authenticator. Match on `errorKey`.
