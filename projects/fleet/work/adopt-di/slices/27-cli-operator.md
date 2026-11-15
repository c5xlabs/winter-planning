---
name: cli-operator
depends-on:
  - hub-request-pilot
---

# Operator command containers

Delivers the operator command boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports`, `engineering:born-in-container`, and `engineering:graph-checked-whole` under
`decision:adopt-di/strangler-migration`.

## Change

- Give client-only commands under `cli/` command-local area roots using existing `CliCollaborators`, context and trace
  inputs; the developer invariant command's client/offline dispatch has its own owner.
- Keep command loading lazy; supply carrier overrides before validation and resolve each command's named collaborators.
- Land command wiring exemplar and transitional CLI guidance with the first usable command; retire replaced
  constructors.

## Acceptance

- `CliRunner` tests preserve invoker-supplied HTTP/context/trace seams and close owned clients on success/failure.
- Fresh-interpreter loaded-module tests show client-only operator commands do not import daemon app/store stacks.
- An exemplar/test-only service is added and consumed through the command path without a per-service root edit.
- Every selected command graph validates; strict typing, relevant CLI tests and `blizzard:cli-contract` pass unchanged.
