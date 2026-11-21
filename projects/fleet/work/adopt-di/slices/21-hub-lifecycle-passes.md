---
name: hub-lifecycle-passes
depends-on:
  - hub-identity
  - hub-cadence
  - hub-events
  - hub-trace-export
code:
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/composition.py::TenantServicesOpener
  - blizzard/src/blizzard/hub/app.py::build_hosted_app
  - blizzard/src/blizzard/hub/app.py::create_app
  - blizzard/src/blizzard/hub/api/hub_level.py::hub_services
  - blizzard/src/blizzard/hub/open_tenants.py::OpenTenants
  - blizzard/src/blizzard/hub/api/sweeps.py::run_every_tenant_pass
  - blizzard/src/blizzard/hub/api/admin_tenants.py::create_tenant
---

# Hub lifecycle, tenancy and pass composition

Moves the hub-lifecycle/pass row in spec [Mixed ownership](../spec/migration.md#area-ownership), using
[Scope ownership](../spec/scopes.md#flows) and typed background access. Delivers `engineering:area-wiring`,
`engineering:ordered-teardown`, and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register `ReadinessService` and its store status reader, action-loop/wake/pass coordination, tenant
  administration/teardown, sweep roster and process pass factories; the platform-tracing keys stay with the scope cache.
- Narrow the sweeps API (`hub/api/sweeps.py`) and the tenant administration API (`hub/api/admin_tenants.py`) from the
  `Hub` bag to the sweep roster, tenant administration and `OpenTenants` services they use.
- Adopt `OpenTenants`, replacing the legacy leaf the request pilot registered, and `ActionLoop` and `StepWake`,
  replacing configured adapters' legacy leaves.
- Register `CloseIntentDrainer` for the close drain, reusing the registered `LandedCommitLinks`.
- Register the `ControlledClock` control routers and their lock, present only when the process clock is controlled.
- Replace remaining generic tenant openers, registry/host lookups and boot callbacks with typed service/read accessors;
  narrow hub-level dependency helpers and hosted-app/driver arguments.
- Move the remaining `HubCore.every_open` stores-only readers, hub boot included, onto `TenantContainers.every_open`.
- Keep app startup/shutdown as lifecycle orchestration over named owners, not a constructor list.
- Remove all newly unused lifecycle fields, legacy factory branches and aggregate accessors.

## Acceptance

- `blizzard:component-test` exercises boot, sweeps, close drain, tenant deletion, and poisoned-tenant isolation through
  new accessors.
- Shutdown/failure tests preserve configured-client/action-loop ordering; the container closes before `hub host`
  disposes the borrowed store and tracing, and never disposes them itself.
- App/driver/auth/helper inventory has no remaining unassigned service-bag consumer; `blizzard:component-test`,
  `blizzard:compat`, `blizzard:typecheck` and the graph/route gates pass, with existing API behavior unchanged.
