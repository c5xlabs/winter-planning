---
name: hub-trace-export
depends-on:
  - hub-events
  - hub-fleet
code:
  - blizzard/src/blizzard/hub/composition.py::build_services
  - blizzard/src/blizzard/hub/domain/observability/tracing/repository.py::StepHydration
  - blizzard/src/blizzard/hub/composition.py::HubServices
  - blizzard/src/blizzard/hub/api/egress.py::egress_status
---

# Trace and egress export providers and consumers

Moves the trace and egress half of the observability row in spec [Mixed ownership](../spec/migration.md#area-ownership),
delivering `engineering:area-wiring`, `engineering:narrow-api-edge`, and `engineering:ordered-teardown` under
`decision:adopt-di/declared-cutover`.

## Change

- Register `TraceStore` and its `StepHydration` accessor, which reads each tenant's registered `WorkRefLabel` and the
  work-source registry's source labels through typed tenant accessors instead of opening `TenantServices`.
- Register the trace export sweep, trace status and replay readers, and the egress sweep, status, reset and backfill
  services, with the egress writers and their path key, over the registered `HubEventLogService`. A disabled export
  degrades today (`trace_export` and `egress_export` are `None`), so each is registered with `Registry.optional` and its
  consumers take `T | None`; it is not an absent registration.
- Narrow the trace, trace-continuation and egress routes and the export sweeps' pass inputs to named services.
- Remove the trace and egress `HubServices` fields and their construction branches.

## Acceptance

- `blizzard:trace-contract` and `blizzard:egress-contract` pass unchanged.
- `blizzard:component-test` over the export sweeps and the trace and egress routes preserves enabled, disabled and
  missing-path-key behavior, and the sweeps' writers close on their existing owner.
- No migrated route takes a bag; `blizzard:typecheck` and the graph/route/identity gates pass.
