---
name: hub-urgency
depends-on:
  - hub-configured-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Urgency and tenant-setting wiring

Moves the urgency portion of spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register `TenantDefault`, urgency read/authoring/lifecycle/ordering/references, and tenant-setting services.
- Register `RoutineUrgencyClearing` and `CadenceUrgencyClearing`, which `UrgencyLifecycle` takes, as their single owner;
  cadence reuses them. Their `IWriteRoutineRepository` and `IWriteCadenceRepository` enter as legacy leaf providers that
  garden authoring and cadence adopt, and the chunk ports they read as legacy leaves that chunk operations adopts.
- The urgencies API takes the registered `ChunkChanged` and `load_frame_states` from configured adapters.
- Narrow urgency and tenant-setting API consumers; retain typed canonical dependencies for unmigrated cadence/routines.
- Remove each old construction branch and newly unused field.

## Acceptance

- Urgency/settings tests preserve default selection, ordering, references and independent tenant state.
- Services are not constructed in the legacy store/service builders; aliases and remaining bag fields share identity.
- `blizzard:component-test`, `blizzard:sse-contract`, `blizzard:typecheck` and the graph/route guards pass with
  unchanged behavior.
