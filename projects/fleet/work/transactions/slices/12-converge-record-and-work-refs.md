---
name: converge-record-and-work-refs
external: github:paul-gross/blizzard#2159
depends-on:
  - store-judgments
  - config-judgments
---

# Converge a chunk's record and work refs

Moves the writes of a chunk's record and work refs onto the one write entry and deletes their `*_locked` twins. Delivers
`engineering:one-write-entry` for this area; spec
[areas.md](../spec/areas.md#record-and-work-refs-converge-record-and-work-refs) and
[write-entry.md](../spec/write-entry.md).

## Change

- Each plain write in the area takes its `transaction` parameter, or a `LockedTransaction` with no default where it is
  correct only under a lock.
- The area's `*_locked` twins, and the locked-handle reads it used, are deleted.
- Lock sites that switch to `IHubTransactions.begin` here: `IngestService.ingest` and `EditService.edit`.
- A cross-area site that calls the area keeps its legacy opener and hands the converted methods `handle.transaction`,
  the transitional form.
- The `work_ref` and `project-source-links` key builders move to the domain.

## Acceptance

- No `*_locked` method of the area remains, and the lock sites above no longer reach the legacy seam.
- Every remaining caller of the area's writes passes a transaction; none calls a twin.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
