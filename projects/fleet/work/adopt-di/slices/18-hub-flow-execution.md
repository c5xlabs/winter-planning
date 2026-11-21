---
name: hub-flow-execution
depends-on:
  - hub-garden-materialization
  - hub-execution
  - hub-graphs
code:
  - blizzard/src/blizzard/hub/composition.py::build_flow_driver
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/api/chunk_events.py::AnnouncedTurns
---

# Flow executor and typed background access

Moves the flow-execution row of spec [Mixed ownership](../spec/migration.md#area-ownership) through the typed background
interfaces in spec [Entrypoint resolution](../spec/entrypoints.md). Delivers `engineering:area-wiring` and
`engineering:scope-identity` under `decision:adopt-di/declared-cutover`.

## Change

- Register hub-node/turn execution and flow-line factories/reconcilers using already-owned collaborators; `TurnRunner`
  takes the platform-tracing `IPlatformTracer` the scope cache registered.
- Own the flow-store aliases `IReadFlows` and `IReadFlowViews` over the registered `FlowStore`, replacing the legacy
  leaves chunk operations' restart, stop, complete and chunk views registered; hub events' invariants reuse them.
- Replace whole-tenant callbacks in flow-driver construction with named typed tenant accessors, reusing the registered
  `HubActions`.
- Narrow `AnnouncedTurns` from `TenantServices` to typed arguments, so `build_flow_driver` no longer wraps turns over
  the bag.
- Narrow the flow-place, flows (`hub/api/flows.py`) and flow-submissions (`hub/api/flow_submissions.py`) handlers and
  flow helpers; remove unused turn/flow bag fields and constructor branches.

## Acceptance

- Flows and flow-submissions routes answer as today (`blizzard:component-test`).
- Flow component tests exercise two tenants through the actual driver accessor and preserve per-row tenant selection.
- Driver and request access share the same tenant-owned dependencies; no domain driver receives a generic container.
- `blizzard:component-test`, `blizzard:sse-contract`, `blizzard:typecheck` and the graph/route/identity gates pass.
- Retired flow branches/fields are absent and remaining legacy references are canonical.
