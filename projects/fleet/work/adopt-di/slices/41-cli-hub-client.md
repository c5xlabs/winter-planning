---
name: cli-hub-client
depends-on:
  - cli-operator
code:
  - blizzard/src/blizzard/hub/cli/command.py::HubCommand
  - blizzard/src/blizzard/hub/cli/collaborators.py::HubCliCollaborators
---

# Hub client-command containers

Delivers the hub client-verb boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports` and `engineering:born-in-container` under `decision:adopt-di/declared-cutover`.

## Change

- Migrate the remaining network/client commands under `hub/cli/` to selected-command area registration.
- Leave the `blizzard hub status` pilot with `cli-operator`; reuse its shared mechanisms without migrating it again.
- Reuse carrier/context/client mechanisms without importing local store/migration/server providers.
- Remove replaced command-root constructors and unused bridges; update affected command exemplars.

## Acceptance

- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) keep daemon/store
  classes out of hub client verbs, including when siblings offer runtime commands; client construction remains lazy
  until requested.
- `blizzard:component-test` (`tests/component/hub/cli`): invoker carrier fakes retain precedence; command
  success/refusal/failure tests preserve outputs and owned cleanup.
- All selected graphs validate; `blizzard:typecheck` and `blizzard:cli-contract` (the command surface) pass unchanged.
- The existing `blizzard hub status` pilot retains its command graph and carrier precedence; no second container or
  provider owner is introduced for it.
