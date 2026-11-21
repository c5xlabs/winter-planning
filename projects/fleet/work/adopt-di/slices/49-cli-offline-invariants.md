---
name: cli-offline-invariants
depends-on:
  - cli-operator
  - hub-events
  - host-infrastructure
code:
  - blizzard/src/blizzard/cli/dev.py::check_invariants_cmd
  - blizzard/src/blizzard/cli/dev.py::_check_offline
  - blizzard/src/blizzard/tools/invariants.py::Invariants
---

# Developer invariant-command containers

Delivers the developer invariant-check boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports`, `engineering:ordered-teardown`, and `engineering:container-only` under
`decision:adopt-di/declared-cutover`.

## Change

- Select client versus explicit offline hub/runner mode before importing mode-specific providers.
- Build the selected command graph from the existing carriers and the bounded invariant/audit capability factories.
  Offline mode builds the owning process's translation as a root the command owns, hub `PROCESS`/`TENANT` or host
  `PROCESS`/`RUNNER` ([Interfaces](../spec/scopes.md#interfaces)), each the one container for its mode, with
  command-local services at that translation's `PROCESS` lifetime; it owns its selected engines and closes the roots on
  exit. Client mode, which has no owning process graph, uses `COMMAND` and loads no store stack.
- Retire replaced helper/manual constructors and update the command's exemplar; keep early compatibility entrypoints
  only if a named non-command caller still needs their registered capability.

## Acceptance

- `blizzard:component-test` (`tests/component/{hub,runner}/cli`) tests preserve running-daemon mode, offline
  hub/runner/both selection, mutually exclusive options, external-database confirmation, after-recovery checks,
  violation reporting and exit codes.
- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) show client mode and
  unrelated operator commands do not import offline provider/store modules.
- `blizzard:component-test`: offline success, invariant failure and partial acquisition failure close every owned engine
  and preserve borrowed inputs.
- The selected graphs validate; an exemplar service can be added within the existing area without a root service edit;
  `blizzard:typecheck` and `blizzard:cli-contract` (the command surface) pass unchanged.
