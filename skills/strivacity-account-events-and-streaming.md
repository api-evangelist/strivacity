---
name: strivacity-account-events-and-streaming
description: Read Strivacity account events and audit entries through the Admin API, and configure outbound event streaming to Splunk or Elasticsearch — including how to tell agent-driven activity from customer-driven activity.
api: Strivacity Admin API
operations:
  - getDescriptor
  - getAccountEvents
  - getAccountEventById
  - getNotificationEvents
  - getNotificationEventById
  - list_4
  - create_6
  - getById
  - update_5
  - delete_7
  - sendTestEvent
generated: '2026-08-29'
method: generated
source: openapi/strivacity-admin-portal-openapi.yml, https://docs.strivacity.com/docs/event-streaming
---

# Account events and event streaming

Strivacity has no HTTP webhook subscription. It has a pull API and a push-to-data-platform integration. Choose by whether you want to poll or to land events in a SIEM.

## Discover the event vocabulary first

`getDescriptor` — `GET /admin/api/v1/events/descriptor` returns the descriptors for account events in this instance. Read it before writing any filter or matcher. It is the closest thing to a published event schema, and instance configuration affects it.

## Pull

- `getAccountEvents` — `GET /admin/api/v1/events/account`. Paginated with `page`/`size`/`sort`, filterable on `startDate`, `endDate`, `accountIds`, `accountEventIds`, `identityStores`, `organizationScope` and more. Requires `read:event_account`.
- `getAccountEventById` — `GET /admin/api/v1/events/account/{eventId}`
- `getNotificationEvents` / `getNotificationEventById` for notification delivery events.

**Agent attribution.** When an AI agent acts on a customer's account, the event's **subject** is the account and the **actor** is the agent's OAuth client id. That pair is how you separate agent-driven activity from activity the customer performed themselves — in the events, in the streamed payload, and in the customer's My Account portal.

## Push

Event streaming forwards audit logs and account events to a vendor destination. Supported destinations are **Splunk** and **Elasticsearch** — it is a vendor list, not an arbitrary callback URL.

1. `list_4` — `GET /admin/api/v1/instance/eventStreaming` (requires `read:config_instance`)
2. `create_6` — `POST /admin/api/v1/instance/eventStreaming`. Choose the vendor, enable audit-log streaming and/or account-event streaming, optionally enable *detailed account event actions*, and name the native claims to include in the payload.
3. `sendTestEvent` — `POST /admin/api/v1/instance/eventStreaming/{id}/test`. **Always do this before declaring the integration live.** It is the only dry-run affordance on this surface.
4. `getById`, `update_5`, `delete_7` to read, change and remove a configuration.

Delivery retries for up to 60 seconds by default, and retries only resume after five events have delivered successfully — a destination that stays down will drop events rather than queue them indefinitely. Retry behaviour is instance-configurable, so confirm it per instance rather than assuming the default.

`delete_7` has no undo. Read the configuration with `getById` and keep it before deleting.

## Note on operation ids

Several operations on this surface carry generated ids — `list_4`, `create_6`, `update_5`, `delete_7`, `getById`. They are unique within the specification but they are not self-describing. Bind by path, not by name, and never guess that `create_6` in some other context means the same thing.
