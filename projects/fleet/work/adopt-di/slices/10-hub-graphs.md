---
name: hub-graphs
depends-on:
  - hub-store-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/delivery/action_registry.py::build_hub_actions
---

# Graph services and packaged inputs

Moves the graphs row of spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register graph mint/default/lifecycle services, codec and packaged artifact inputs and graph read aliases.
- Register `HubActions` (`build_hub_actions`) at process lifetime; garden authoring, flow execution and hub lifecycle
  reuse it.
- Narrow graph/system-artifact API and helper consumers without changing configuration or packaged graph behavior.
- Remove unused graph bag fields; legacy execution receives the same registered graph dependencies.

## Acceptance

- Existing graph lifecycle, default-graph and packaged-artifact tests pass for both tenant and hub-only consumers.
- Explicit factories preserve defaults; graph aliases and any retained bag references share identity.
- `blizzard:component-test`, `blizzard:typecheck` and the graph/route gates pass; old owned constructors are gone.
