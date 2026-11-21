---
name: hub-garden-authoring
external: github:paul-gross/blizzard#2372
depends-on:
  - hub-operations
  - hub-projects
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Garden authoring services

Moves the garden-authoring row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register scope/routine lifecycle and authoring, scope membership, baselines, run context/answered findings, proposal
  authoring/closure, proposal-delivery resolution and `GardenStageCheck`, reusing the registered `HubActions`.
- Register `GardenFormats`, whose first consumer is `GardenStageCheck`; garden materialization and garden runs reuse it.
- Own the finding-set aliases (`IReadFindingSetRepository` and its writer), which scopes and routines read first;
  materialization, runs and cadence reuse them.
- Adopt `IWriteRoutineRepository`, replacing urgency's legacy leaf.
- Narrow scope, routine and proposal handlers/read helpers; preserve the finding providers already adopted.
- Remove old construction branches and newly unused bag fields; keep materialization/run dependencies canonical.

## Acceptance

- Scope/routine/proposal component/API tests preserve transaction and finding/proposal lifecycle behavior.
- No migrated handler/helper takes a bag; each service has one registration and shared dependency identity.
- `blizzard:component-test`, `blizzard:typecheck` and the graph/route gates pass; existing external behavior is
  unchanged.
