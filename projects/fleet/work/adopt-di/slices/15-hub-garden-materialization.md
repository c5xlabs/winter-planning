---
name: hub-garden-materialization
depends-on:
  - hub-garden-authoring
  - hub-configured-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Garden delivery materialization

Moves the garden-materialization row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:behavior-preserving-landings`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register garden/review delivery recorders and materializers, reusing the registered `GardenFormats` and finding-set
  aliases.
- Pass named collaborators to the remaining flow executor construction; never obtain them through a service bag.
- Remove the replaced constructor branches and any newly unused materialization bridges.

## Acceptance

- Delivery/review component tests preserve materialized findings, proposals, facts and commit-resolution behavior.
- The flow path and legacy consumers use the canonical recorder/materializer instances without an aggregate cycle.
- `blizzard:component-test`, `blizzard:typecheck` and the graph/ownership gates pass with unchanged external behavior.
