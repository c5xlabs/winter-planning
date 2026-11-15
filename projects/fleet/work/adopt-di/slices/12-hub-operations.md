---
name: hub-operations
depends-on:
  - hub-execution
  - hub-urgency
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
---

# Chunk operations and read consumers

Moves the chunk-operations row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring`, `engineering:narrow-api-edge`, and `engineering:behavior-preserving-landings` under
`decision:adopt-di/declared-cutover`.

## Change

- Register remaining operations, ingest/dependency/work-item-edit services and event log, preserving constructor
  defaults.
- Narrow chunk edit/dependency/read/view/lifecycle and queue/ingest helpers, including `ChunkReadStores` consumers;
  leave the `chunk_statuses` API and event/export status handlers with observability.
- Remove retired fields/branches; keep any remaining background bag references canonical.

## Acceptance

- Relevant operation and read component/API tests preserve state transitions, tenant filtering and event publication.
- No affected route/helper/service takes a bag; retained bridge references equal their canonical providers.
- Strict typing and graph/route/ownership gates pass; planted bag dependencies fail the migrated-area guard.
