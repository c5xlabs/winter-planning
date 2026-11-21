---
name: hub-garden-runs
depends-on:
  - hub-garden-authoring
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Garden trend, sweep and run services

Moves the garden-runs row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register trend/sweep/run and routine run services with their named dependencies and explicit default factories;
  `GardenRunService` reuses the registered `GardenFormats` and finding-set aliases.
- Narrow garden-run handlers and remaining garden-run read helpers.
- Remove old constructor branches and unused run fields; keep cadence callers on canonical instances.

## Acceptance

- Garden trend/run/sweep component/API tests preserve planning, grouping and tenant behavior.
- No affected handler/helper takes a bag; cadence/legacy references retain identity with registered services.
- `blizzard:component-test`, `blizzard:typecheck` and the graph/route gates pass; available run bridges are removed.
