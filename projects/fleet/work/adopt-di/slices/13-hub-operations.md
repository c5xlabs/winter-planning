---
name: hub-operations
depends-on:
  - hub-execution
  - hub-urgency
code:
  - blizzard/src/blizzard/hub/composition.py::build_tenant_stores
  - blizzard/src/blizzard/hub/composition.py::build_tenant_services
  - blizzard/src/blizzard/hub/store/internal/work_item_store.py::WorkItemStore
  - blizzard/src/blizzard/hub/api/work_item_reads.py::read_chunk_work_items
---

# Chunk operations and read consumers

Moves the chunk-operations row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring`, `engineering:narrow-api-edge`, and `engineering:behavior-preserving-landings` under
`decision:adopt-di/declared-cutover`.

## Change

- Register the `hub/domain/operations/` services (promote, demote, restart, pause, stop, complete, edit, delete, queue,
  group, claim explanation), `hub/domain/chunk/`'s `IngestService` and `DependencyService`, `hub/domain/work_items/`'s
  `WorkItemEditService`, and the event log, preserving constructor defaults.
- Own the chunk-port aliases (`IRead`/`IWriteChunk*` over `build_chunk_stores`), replacing the legacy leaves configured
  adapters, urgency, fleet and execution registered.
- Own the work-item port aliases (`IReadWorkItemRepository`/`IWriteWorkItemRepository` over the registered
  `WorkItemStore`) that ingest, cadence mint and the garden run service take, replacing the legacy leaves configured
  adapters' work-source registry and earlier slices registered.
- Narrow `read_chunk_work_items` and the work-item-reads router (`hub/api/work_item_reads.py`) to the work-item and
  chunk reads they use.
- Narrow chunk edit/dependency/read/view/lifecycle and queue/ingest helpers, including `ChunkReadStores` consumers;
  leave the `chunk_statuses` API and event/export status handlers with observability.
- Remove retired fields/branches; keep any remaining background bag references canonical.

## Acceptance

- `blizzard:component-test` over operation and read routes preserves state transitions, tenant filtering and event
  publication; `blizzard:sse-contract` passes.
- No affected route/helper/service takes a bag; retained bridge references equal their canonical providers.
- `blizzard:typecheck` and the graph/route/ownership gates pass; planted bag dependencies fail the migrated-area guard.
