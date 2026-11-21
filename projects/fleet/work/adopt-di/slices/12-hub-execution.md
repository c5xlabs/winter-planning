---
name: hub-execution
depends-on:
  - hub-fleet
  - hub-configured-adapters
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/api/ingest_broadcast.py::IngestBroadcast
---

# Chunk execution services

Moves the remaining execution row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:narrow-api-edge`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register claim/apply/decision/release/requeue/question and fact/host-fact services; reuse adopted detach/fleet guards.
- Narrow their runner-contact and chunk handlers, including dependency functions, not unrelated handlers in those files.
  Claim and apply take the `WorkRefLabel` configured adapters registered.
- Narrow `IngestBroadcast`, which the fleet API constructs over `TenantServices`, to the typed reads and publisher it
  uses.
- Use canonical legacy leaf providers for any unmoved dependency; remove old constructors and unused fields.

## Acceptance

- Claim/apply/question/fact/refusal component tests preserve transaction, event and supported-client behavior.
- Changed handlers name exact services; shared guards, event dependencies and retained bag fields share identity.
- `blizzard:component-test`, `blizzard:compat`, `blizzard:typecheck` and the graph/route gates pass; no legacy aggregate
  is a provider's dependency source.
