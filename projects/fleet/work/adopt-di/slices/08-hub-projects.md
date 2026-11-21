---
name: hub-projects
depends-on:
  - hub-configured-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Project services and read helpers

Moves the project portion of the projects/urgency row in spec [Mixed ownership](../spec/migration.md#area-ownership),
delivering `engineering:area-wiring` and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register project authoring, bulk project resolution, chunk targets and `ChunkTargetCapture` (consumed by
  `ClaimService` and `FlowLineReconciler`, which take the registered provider when they migrate), the
  `IReadProjectRepository` alias (adopting the request pilot's legacy leaf), project-link/position read aliases, and
  color inputs, `ColorPick` among them (adopting the configured adapters' legacy leaf).
- Narrow project handlers, parameter resolution and label helpers; existing bag callers pass explicit helper arguments.
- Remove newly unused project fields and constructor/bridge branches.

## Acceptance

- Project/label component tests preserve project-lens filtering and unknown-project behavior in two tenants.
- Migrated helpers and routes name their services/read ports; aliases and retained legacy fields share canonical
  instances.
- `blizzard:component-test`, `blizzard:typecheck` and the graph/route/ownership guards pass with unchanged wire
  behavior.
