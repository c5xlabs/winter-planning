---
name: hub-flow-execution
depends-on:
  - hub-garden-materialization
  - hub-execution
  - hub-graphs
code:
  - blizzard/src/blizzard/hub/composition.py::build_flow_driver
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Flow executor and typed background access

Moves the flow-execution row of spec [Mixed ownership](../spec/migration.md#area-ownership) through the typed background
interfaces in spec [Entrypoint resolution](../spec/entrypoints.md). Delivers `engineering:area-wiring` and
`engineering:scope-identity` under `decision:adopt-di/declared-cutover`.

## Change

- Register hub-node/turn execution and flow-line factories/reconcilers using already-owned collaborators.
- Replace whole-tenant callbacks in flow-driver construction with named typed tenant accessors.
- Narrow flow-place handlers and flow helpers; remove unused turn/flow bag fields and constructor branches.

## Acceptance

- Flow component tests exercise two tenants through the actual driver accessor and preserve per-row tenant selection.
- Driver and request access share the same tenant-owned dependencies; no domain driver receives a generic container.
- Existing flow/turn API and component tests, strict typing and graph/route/identity gates pass.
- Retired flow branches/fields are absent and remaining legacy references are canonical.
