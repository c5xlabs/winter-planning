---
name: hub-cadence
depends-on:
  - hub-garden-runs
  - hub-urgency
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Cadence and errand planning

Moves the cadence row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register cadence authoring/lifecycle/read/mint, attention/urgency clearing, errand planning/timeline and tenant
  cadence.
- Narrow cadence API and its helper dependencies; expose a typed tenant cadence accessor to the process sweep.
- Remove replaced constructors and unused bag fields, leaving the process sweep's own lifecycle for its owner.

## Acceptance

- Cadence/errand component/API tests preserve scheduling and per-tenant failure isolation.
- HTTP and background access resolve the same tenant cadence dependencies without a synthetic request/principal.
- Strict typing and graph/route/identity gates pass; available cadence bridges are removed.
