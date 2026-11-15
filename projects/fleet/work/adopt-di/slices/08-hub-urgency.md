---
name: hub-urgency
depends-on:
  - hub-store-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Urgency and tenant-setting wiring

Moves the urgency portion of spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge` under `decision:adopt-di/declared-cutover`.

## Change

- Register `TenantDefault`, urgency read/authoring/lifecycle/ordering/references, and tenant-setting services.
- Narrow urgency and tenant-setting API consumers; retain typed canonical dependencies for unmigrated cadence/routines.
- Remove each old construction branch and newly unused field.

## Acceptance

- Urgency/settings tests preserve default selection, ordering, references and independent tenant state.
- Services are not constructed in the legacy store/service builders; aliases and remaining bag fields share identity.
- Relevant component/API tests, strict typing and graph/route guards pass with unchanged behavior.
