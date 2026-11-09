---
name: store-held-exclusions
depends-on:
  - converge-chunk-dependencies
  - converge-graph
---

# Store-held exclusions

Moves the dependency graph's cycle check, the delete and release that share its lock, and the export cursor's pass lock
onto locks the store holds. Delivers `engineering:store-held-exclusions`; waits on
`decision:transactions/cycle-check-lock`. Spec `Data`.

## Change

- The cycle check, dependency delete, and dependency release take a store lock in place of the per-tenant in-process
  lock.
- The export cursor's reset and pass take a store lock in place of the in-process lock.
- A lock needing a store row no table holds lands as an expansion; a `keyed_locks` row needs no revision.
- The sweep locks stay in process.

## Acceptance

- Neither in-process lock remains.
- Two concurrent dependency changes that would together close a cycle: one is refused.
- An operator's reset of the export cursor is not overwritten by a pass in flight.
