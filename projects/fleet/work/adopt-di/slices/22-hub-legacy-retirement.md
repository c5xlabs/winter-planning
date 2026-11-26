---
name: hub-legacy-retirement
external: github:paul-gross/blizzard#2367
depends-on:
  - hub-lifecycle-passes
  - hub-offline-roots
code:
  - blizzard/src/blizzard/hub/composition.py::HubServices
  - blizzard/src/blizzard/hub/composition.py::TenantServices
  - blizzard/src/blizzard/hub/composition.py::TenantStores
  - blizzard/src/blizzard/hub/composition.py::HubCore
  - blizzard/src/blizzard/hub/composition.py::TenantServiceParts
  - blizzard/src/blizzard/hub/composition.py::ServedTenant
  - blizzard/src/blizzard/hub/composition.py::TenantSweeps
  - blizzard/src/blizzard/hub/composition.py::TenantServing
  - blizzard/src/blizzard/hub/composition.py::LiveConfig
  - blizzard/src/blizzard/hub/composition.py::TenantServicesOpener
  - blizzard/src/blizzard/hub/composition.py::TenantCache
  - blizzard/src/blizzard/hub/composition.py::IOpenTenantServices
  - blizzard/src/blizzard/hub/domain/chunk/ports/stores.py::ChunkReadStores
  - blizzard/src/blizzard/hub/api/deps.py::Services
  - blizzard/src/blizzard/hub/api/deps.py::tenant_services
  - blizzard/src/blizzard/hub/api/hub_level.py::Hub
  - blizzard/src/blizzard/hub/api/auth_session.py::TenantRequest
---

# Close the hub's legacy registration

Closes the hub portion of spec [Mixed ownership](../spec/migration.md#flows), delivering `engineering:container-only`
and `engineering:rules-follow-wiring` under `decision:adopt-di/declared-cutover`. Field and branch removal had not
finished with the earlier slices: six route handlers and the shared test harness still read the bags, so this closure
also moves those leftover consumers onto the container before it deletes the bags.

## Change

- Remove empty hub legacy registration, the aggregate types and assemblers spec
  [Mixed ownership](../spec/migration.md#domain) retires (`HubCore`, `TenantServing` and `LiveConfig` among them), the
  legacy `TenantCache`, `_TenantEntry` and `IOpenTenantServices` with the request bridge (`deps.Services`,
  `deps.tenant_services`, `hub_level.Hub`, and `TenantRequest`'s `services` field), and transition allowances, after
  proving no consumers. `HubSettingsGraph` stays: it is borrowed context `hub host` supplies.
- Make the hub's area-root, no-bag, registered-construction and typed-accessor guards container-only.
- Bring hub harness rules/exemplars to their end state while explicitly retaining unmigrated runner/CLI guidance.

## Acceptance

- A source/inventory gate finds no hub legacy provider, aggregate construction or services-bag dependency; planted
  violations fail rather than being added to an allowlist.
- Offline hub roots run through their compatibility entrypoints; none constructs `HubCore` or reads a `TenantStores`
  field.
- Hub roots contain area composition and lifecycle orchestration, not individual service registrations.
- Whole-graph and every-route tests, `blizzard:typecheck`, `blizzard:component-test`, `blizzard:service-test`,
  `blizzard:compat` and `blizzard:config-contract` pass unchanged; hub CLI commands behave as before
  (`blizzard:component-test`), and the command surface is unchanged (`blizzard:cli-contract`).
- The hub `bzh:` rules state the container-only hub and pass `blizzard-context:reference-lint` and
  `blizzard-context:registry-drift`; the hub exemplars pass `blizzard:typecheck`, `blizzard:unit-test` and
  `blizzard:component-test`.
- If an aggregate still has a live consumer, its owning migration change is incomplete and this closure does not pass;
  no builder is removed first while its bag remains needed.
