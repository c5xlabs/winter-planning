---
name: hub-cadence
external: github:paul-gross/blizzard#2364
depends-on:
  - hub-garden-runs
  - hub-urgency
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/api/chunk_events.py::announce_minted
---

# Cadence and errand planning

Moves the cadence row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register cadence authoring/lifecycle/read/mint, attention, errand planning and tenant cadence, reusing the urgency
  clearings and finding-set aliases already registered; adopt the cadence store ports, `IWriteCadenceRepository` among
  them, replacing urgency's legacy leaf.
- Narrow `announce_minted` from `TenantServices` to typed arguments, so `TenantCadence`'s `announce` callback no longer
  closes over the bag.
- Narrow cadence API and its helper dependencies; expose a typed tenant cadence accessor to the process sweep.
- Remove replaced constructors and unused bag fields, leaving the process sweep's own lifecycle for its owner.

## Acceptance

- Cadence/errand component/API tests preserve scheduling and per-tenant failure isolation.
- HTTP and background access resolve the same tenant cadence dependencies without a synthetic request/principal.
- `blizzard:component-test`, `blizzard:sse-contract`, `blizzard:typecheck` and the graph/route/identity gates pass;
  available cadence bridges are removed.
