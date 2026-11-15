---
name: hub-observability
depends-on:
  - hub-operations
  - hub-flow-execution
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/composition.py::build_hub_invariants
---

# Observability providers and consumers

Moves the observability row of spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring`, `engineering:narrow-api-edge`, and `engineering:ordered-teardown` under
`decision:adopt-di/declared-cutover`.

## Change

- Register transcript/analytics, tenant event derivation/annotation, trace/egress readers/exporters and invariant
  services.
- Narrow event/chunk-event/status/export/invariant consumers; give process passes typed tenant accessors.
- Adapt `tools.invariants.hub_invariants` to the registered bounded invariant capability before old hub roots retire,
  preserving its callable interface for the not-yet-migrated offline developer command.
- Preserve disabled optional services, role-distinct clocks, stream broker isolation and owner-loop finalizers.
- Remove old constructors and unused fields; leave process pass scheduling/roster assembly to its lifecycle owner.

## Acceptance

- Relevant transcript/event/trace/egress/invariant tests preserve disabled/enabled cases and resource ownership.
- SSE isolation tests show no cross-tenant fan-out/replay and closing one tenant leaves the other's streams intact.
- Export/trace contracts, affected component/API tests, strict typing and graph/route/identity gates pass.
- No migrated consumer accepts a bag; available observability bridges are removed.
- The existing offline hub invariant command still passes using the compatibility entrypoint, without importing a
  retired hub aggregate.
