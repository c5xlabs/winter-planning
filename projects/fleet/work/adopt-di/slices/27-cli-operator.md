---
name: cli-operator
depends-on:
  - hub-request-pilot
code:
  - blizzard/src/blizzard/cli/operator_trace.py::OperatorGroup.invoke
  - blizzard/src/blizzard/hub/cli/command.py::HubCommand.invoke
  - blizzard/src/blizzard/hub/cli/status.py::status
---

# Operator command containers

Builds shared command-container support and activates the `blizzard hub status` pilot in spec
[Entrypoint resolution](../spec/entrypoints.md), serving `engineering:selective-command-imports`,
`engineering:born-in-container`, and `engineering:graph-checked-whole` under `decision:adopt-di/strangler-migration`.

## Change

- Build shared command-local client/context/trace registration support using existing carrier inputs, and migrate
  `blizzard hub status` to that path.
- Keep command loading lazy; supply carrier overrides before validation and resolve the status command's named
  collaborators without activating sibling command graphs.
- Land the command wiring exemplar and transitional CLI guidance with this usable pilot; retire its replaced
  constructors. Other command bodies retain their existing wiring.

## Acceptance

- `CliRunner` tests invoke `blizzard hub status`, preserve invoker-supplied HTTP/context/trace seams, and close owned
  clients on success/failure with unchanged output and exit status.
- Fresh-interpreter loaded-module tests show the status command imports neither daemon app/store stacks nor sibling
  command providers.
- An exemplar/test-only service is added and consumed through the status command's production resolution path without a
  per-service root edit.
- The pilot graph validates; strict typing, relevant CLI tests and `blizzard:cli-contract` pass unchanged, including
  still-unconverted commands.

## Out of scope

- Remaining hub client verbs migrate in `cli-hub-client`; hub local verbs migrate in `cli-hub-local`.
- Runner worker and local verbs migrate in `cli-worker` and `cli-runner-local`.
- The developer invariant command's client/offline dispatch migrates in `cli-offline-invariants`.
