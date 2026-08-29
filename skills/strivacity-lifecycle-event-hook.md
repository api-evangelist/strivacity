---
name: strivacity-lifecycle-event-hook
description: Author, try out, deploy and troubleshoot a Strivacity Lifecycle Event Hook — the sandboxed JavaScript that runs at a named point in a customer's login, registration or account lifecycle.
api: Strivacity Admin API
operations:
  - getAllEventHooks
  - getEventHooks
  - getSnippets
  - getSnippetCode
  - getSuggestions
  - getPackage
  - createEventHookFunction
  - getEventHookFunctionById
  - updateEventHookFunction
  - createFunctionResource
  - callFunctionResource
  - deleteFunctionResource
  - deployEventHookFunction
  - getEventHookFunctionStatusById
  - getEventHookLogs
  - getEventHookLogById
  - deleteEventHookFunction
generated: '2026-08-29'
method: generated
source: openapi/strivacity-admin-portal-openapi.yml, https://docs.strivacity.com/docs/lifecycle-event-hooks
---

# Lifecycle event hooks

A Lifecycle Event Hook is brand-authored JavaScript that Strivacity executes inside a sandboxed serverless environment at a named point in the customer lifecycle. **Synchronous** hooks take control and can alter or block the flow; **asynchronous** hooks run alongside it and cannot.

There are 24 documented hook points, split into *at failed…*, *after…* and *before…* families. `before-id-token-generation` can inject claims and add or remove scopes; `before-contact-update` can block an email or phone change; `after-account-deletion` is how deletions get synced outward.

## Steps

1. **Discover the hook points.** `getAllEventHooks` — `GET /admin/api/v1/eventHooks/descriptor`. Requires `read:config_event_hook`.
2. **Start from a template.** `getEventHooks` — `GET /admin/api/v2/pluginLibrary/eventHooks` lists the plugin library; `getSnippets` / `getSnippetCode` (`GET /admin/api/v2/pluginLibrary/eventHookSnippets/{type}[/{id}]`) return code for a given hook type. Templates arrive with their dependencies and their entry point already configured.
3. **Resolve dependencies.** `getSuggestions` — `POST /admin/api/v1/eventHooks/npm/search` filters NPM dependency suggestions; `getPackage` — `POST /admin/api/v1/eventHooks/npm/package` lists versions for one. Pin a version.
4. **Create.** `createEventHookFunction` — `POST /admin/api/v1/eventHooks/function`. Requires `write:config_event_hook`.
5. **Try it out before deploying.** `createFunctionResource` — `POST /admin/api/v1/eventHooks/function/{functionId}/test`, then `callFunctionResource` — `POST .../test/{id}/call` to execute against sample input, then `deleteFunctionResource` to clean up. This is the rehearsal step; a synchronous hook that throws in production blocks a real customer's login.
6. **Deploy.** `deployEventHookFunction` — `POST /admin/api/v1/eventHooks/function/{id}/deploy`, then poll `getEventHookFunctionStatusById` — `GET .../function/{id}/status`.
7. **Assign** the hook to an application or identity store. Until it is assigned it does not run.
8. **Troubleshoot.** `getEventHookLogs` — `GET /admin/api/v1/eventHooks/logs` and `getEventHookLogById` require `read:event_hook_log`. Account event details link straight to the related hook execution logs.

## Deleting

`deleteEventHookFunction` — `DELETE /admin/api/v1/eventHooks/function/{id}`. A hook that is currently **assigned** cannot be deleted; unassign it first. There is no restore, so export the code before deleting.

## Listing caveat

On `getAllEventHookFunctions` (`GET /admin/api/v1/eventHooks/function`) the `policyTags` query parameter is **ignored** whenever the `hook` query parameter is also sent. Send one or the other.
