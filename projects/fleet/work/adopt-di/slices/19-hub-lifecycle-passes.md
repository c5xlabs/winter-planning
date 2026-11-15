---
name: hub-lifecycle-passes
depends-on:
  - hub-identity
  - hub-cadence
  - hub-observability
code:
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/composition.py::TenantServicesOpener
---

# Hub lifecycle, tenancy and pass composition

Moves the hub-lifecycle/pass row in spec [Mixed ownership](../spec/migration.md#area-ownership), using
[Scope ownership](../spec/scopes.md#flows) and typed background access. Delivers `engineering:area-wiring`,
`engineering:ordered-teardown`, and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register action-loop/wake/pass coordination, tenant administration/teardown, sweep roster and process pass factories.
- Replace remaining generic tenant openers, registry/host lookups, minted announcements and boot callbacks with typed
  service/read accessors; narrow hub-level dependency helpers and hosted-app/driver arguments.
- Keep app startup/shutdown as lifecycle orchestration over named owners, not a constructor list.
- Remove all newly unused lifecycle fields, legacy factory branches and aggregate accessors.

## Acceptance

- Component tests exercise boot, sweeps, close drain, tenant deletion, and poisoned-tenant isolation through new
  accessors.
- Shutdown/failure tests preserve configured-client/action-loop/store ordering and borrowed input ownership.
- App/driver/auth/helper inventory has no remaining unassigned service-bag consumer; strict typing and graph/route gates
  pass, with existing API and supported-release behavior unchanged.
