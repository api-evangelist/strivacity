---
name: strivacity-provision-customer-account
description: Create, look up, group, role-assign, disable and delete a customer account in a Strivacity identity store through the Admin API — including which of those actions can be taken back.
api: Strivacity Admin API
operations:
  - listAccounts
  - createAccount
  - getAccountById
  - addGroupsToAccount
  - addRolesToAccount
  - getAccountRolesById
  - deleteRolesFromAccount
  - updateAccountOrganization
  - updateAccountIdentifier
  - disableAccount
  - enableAccount
  - deleteAccountById
generated: '2026-08-29'
method: generated
source: openapi/strivacity-admin-portal-openapi.yml
---

# Provision a customer account

All paths are scoped by identity store **name**, not id: `/admin/api/v1/identityStores/{identityStoreName}/accounts`.

## Steps

1. **Check for an existing account first.** `listAccounts` — `GET .../accounts`. Supports `page`, `size`, `sort` plus filters including `loginIdentifier`, `emailNativeClaim`, `userNameNativeClaim`, `enabled`, and created/updated/disabled date ranges. Different parameters AND together; repeated values inside one parameter OR together.

   Do this even on a first attempt. The API has **no idempotency support** — no `Idempotency-Key` header exists in any published spec — so a retried `createAccount` after a timeout returns `409`, not the original result. Listing on a natural key is the only safe reconciliation path.

2. `createAccount` — `POST .../accounts`. Optional request headers change side effects:
   - `X-Skip-Email-Verification` — suppress the email verification step
   - `X-Skip-Phone-Verification` — suppress the phone verification step
   - `X-Notification-By-Application` — attribute the resulting notification to a named application

3. `getAccountById` — `GET .../accounts/{accountId}` to confirm the created state.

4. Attach authorization:
   - `addGroupsToAccount` — `POST .../accounts/{accountId}/groups`
   - `addRolesToAccount` — `POST .../accounts/{accountId}/roles` (organization role assignments)
   - `updateAccountOrganization` — `PUT .../accounts/{accountId}/organization` for B2B placement

## Reversibility — read before you delete

- `disableAccount` (`POST .../accounts/{accountId}/disable`) is **fully reversible** by `enableAccount` (`POST .../accounts/{accountId}/enable`). Prefer it.
- `deleteAccountById` (`DELETE .../accounts/{accountId}`) has **no restore operation and no published retention window**. Strivacity documents no undo period for a deleted account. Treat it as permanent.
- `deleteRolesFromAccount` is reversible by re-assigning the role.

If a request is ambiguous about whether an account should be removed or suspended, choose disable and say so.

## Scopes

`read:account`, `write:account`, `delete:account`. Granted through an API Access policy — see `strivacity-admin-api-access-token`.

## Errors

`application/problem+json`. `400` carries `fieldErrors[]` naming each violating field with `objectName`, `field` and `message`. `409` on create means a uniqueness or state conflict. Match on `errorKey`.
