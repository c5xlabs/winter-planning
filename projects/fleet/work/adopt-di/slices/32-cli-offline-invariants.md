---
name: cli-offline-invariants
depends-on:
  - cli-operator
  - hub-observability
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
- Build the selected command graph from the existing carriers and the bounded invariant/audit capability factories;
  offline mode owns its selected engines, while client mode loads no store stack.
- Retire replaced helper/manual constructors and update the command's exemplar; keep early compatibility entrypoints
  only if a named non-command caller still needs their registered capability.

## Acceptance

- CLI tests preserve running-daemon mode, offline hub/runner/both selection, mutually exclusive options,
  external-database confirmation, after-recovery checks, violation reporting and exit codes.
- Fresh-interpreter tests show client mode and unrelated operator commands do not import offline provider/store modules.
- Offline success, invariant failure and partial acquisition failure close every owned engine and preserve borrowed
  inputs.
- The selected graphs validate; an exemplar service can be added within the existing area without a root service edit;
  strict typing and relevant invariant/CLI tests pass unchanged.
