---
name: cli-hub-client
depends-on:
  - cli-operator
---

# Hub client-command containers

Delivers the hub client-verb boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports` and `engineering:born-in-container` under `decision:adopt-di/declared-cutover`.

## Change

- Migrate the network/client commands under `hub/cli/` to selected-command area registration.
- Reuse carrier/context/client mechanisms without importing local store/migration/server providers.
- Remove replaced command-root constructors and unused bridges; update affected command exemplars.

## Acceptance

- Fresh-interpreter import tests keep daemon/store classes out of hub client verbs, including when siblings offer
  runtime commands; client construction remains lazy until requested.
- Invoker carrier fakes retain precedence; command success/refusal/failure tests preserve outputs and owned cleanup.
- All selected graphs validate; strict typing, affected CLI tests and `blizzard:cli-contract` pass unchanged.
