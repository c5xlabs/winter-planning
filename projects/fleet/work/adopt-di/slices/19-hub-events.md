---
name: hub-events
external: github:paul-gross/blizzard#2365
depends-on:
  - hub-operations
  - hub-flow-execution
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/composition.py::build_hub_invariants
  - blizzard/src/blizzard/hub/domain/observability/hub_events.py::HubEventLogService
  - blizzard/src/blizzard/hub/event_log_opener.py::system_event_log_opener
  - blizzard/src/blizzard/tools/invariants.py::hub_invariants
  - blizzard/src/blizzard/hub/api/spend.py::fleet_spend
  - blizzard/src/blizzard/hub/domain/observability/env_timeline.py::EnvTimelineReading
---

# Event, transcript and invariant providers and consumers

Moves the event, transcript and invariant half of the observability row in spec
[Mixed ownership](../spec/migration.md#area-ownership), delivering `engineering:area-wiring`,
`engineering:narrow-api-edge`, and `engineering:ordered-teardown` under `decision:adopt-di/declared-cutover`.

## Change

- Register transcript/analytics, tenant event derivation/annotation, spend reads and invariant services, reusing the
  `TenantBrokers` and tenant `EventBroker` the scope cache registered, and holding the SSE brokers' tenant isolation.
- Register `EnvTimelineReading` as its single owner, and narrow the env-timeline route (`hub/api/env_timeline.py`) to
  that reader and the transcripts read it takes.
- Register `HubEventLogService`/`IReadHubEventLog` and narrow the admin events API; turn `system_event_log_opener` into
  a bounded compatibility entrypoint for hub init/migrate ([Mixed ownership](../spec/migration.md#area-ownership)).
- Narrow event, chunk-event, status, transcript, analytics, spend and invariant consumers; give process passes typed
  tenant accessors.
- Adapt `tools.invariants.hub_invariants`, which now builds its own connections, to the registered bounded invariant
  capability before old hub roots retire, preserving its callable interface for the not-yet-migrated offline developer
  command.
- Preserve disabled optional services, role-distinct clocks, stream broker isolation and owner-loop finalizers.
- Remove old constructors and unused fields; leave process pass scheduling/roster assembly to its lifecycle owner.

## Acceptance

- `blizzard:component-test` over transcript, event, spend and invariant routes preserves disabled and enabled cases and
  resource ownership.
- `blizzard:sse-contract` and SSE isolation tests show no cross-tenant fan-out or replay, and closing one tenant leaves
  the other's streams intact.
- Hub init/migrate record their system events unchanged through the event-log compatibility entrypoint.
- The existing offline hub invariant command still passes using the compatibility entrypoint, without importing a
  retired hub aggregate.
- No migrated consumer accepts a bag; `blizzard:typecheck` and the graph/route/identity gates pass.
