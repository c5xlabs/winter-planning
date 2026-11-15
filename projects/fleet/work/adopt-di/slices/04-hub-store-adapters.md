---
name: hub-store-adapters
depends-on:
  - hub-request-pilot
code:
  - blizzard/src/blizzard/hub/composition.py::build_hub_core
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Hub store registrations

Moves the scoped-store ownership row of spec [Mixed ownership](../spec/migration.md#area-ownership) into adapter wiring,
delivering `engineering:area-wiring`, `engineering:one-instance-per-port`, and
`engineering:behavior-preserving-landings` under `decision:adopt-di/adoption-batch-size`.

## Change

- Register the remaining hub/store adapters and connection/transaction primitives currently constructed by the three
  listed roots; retain the finding registrations already adopted.
- Replace their old constructor branches with canonical resolutions; services embedded in `TenantStores` remain with
  their concept owners, not the store package.
- Remove proved-dead fields only after checking API, helpers, and background consumers
  (`engineering:dead-members-deleted`).

## Acceptance

- Each scoped-store constructor has one provider owner; alias and legacy-store field identity checks pass.
- Unit/component tests preserve tenant filtering, transaction boundaries, shared engine ownership, and both tenant
  caches' independent lifetimes.
- The graph and current routes resolve; no new concrete store construction remains in the listed roots.
- `blizzard:typecheck`, relevant store/component tests and unchanged API behavior checks pass.
