---
name: strivacity-consent-and-personal-data
description: Read, grant and revoke a customer's consents and export their personal data through Strivacity — the two flows a privacy or DSAR request actually needs.
api: Strivacity MyAccount API
operations:
  - getConsents
  - optInConsent
  - optOutConsent
  - getPersonalData
generated: '2026-08-29'
method: generated
source: openapi/strivacity-myaccount-portal-openapi.yml, openapi/strivacity-admin-portal-openapi.yml
---

# Consent and personal data

Two paths exist for the same underlying data — the customer-facing one and the administrator-facing one. Pick deliberately.

## As the customer (MyAccount API)

1. `getConsents` — `GET /myaccount/api/v1/consents`. Returns the consents assigned to the application together with the customer's current state.
2. `optInConsent` — `POST /myaccount/api/v1/consents/{consentId}/optIn`
3. `optOutConsent` — `POST /myaccount/api/v1/consents/{consentId}/optOut`
4. `getPersonalData` — `GET /myaccount/api/v1/me/personalData`. Downloads the account information. Accepts a `fileType` query parameter.

## As an administrator (Admin API)

- `getPersonalData` — `GET /admin/api/v1/identityStores/{identityStoreName}/accounts/{accountId}/download`. Same export, taken on the customer's behalf. Requires `read:account_download`.

Note that both operations are named `getPersonalData`. They are different operations in different specifications with different paths and different authorization. Do not resolve the name without the spec.

## Reversibility

Consent is **versioned**, and opting out is recorded as a new state rather than erasing the prior grant — consent receipts are retained for audit. This is one of the few Strivacity write surfaces where the history survives the reversal, and it is deliberate: it is what a regulator asks for.

There is no published window on any of this. An opt-out does not expire and cannot be un-done except by a new opt-in.

## Consent versioning rules (Admin API)

When creating or updating the consent *definition* rather than a customer's answer: `createConsent` requires the new consent's version to be `1`, and `updateConsent` requires the version to be unchanged or incremented by exactly one. Skipping a version is rejected.

## Scopes

`read:config_consent`, `write:config_consent` for definitions; `read:account_download` for the administrative export.
