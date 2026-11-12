---
name: converge-hub-execution
depends-on:
  - store-judgments
---

# Converge a chunk's hub execution

Moves the writes of a chunk's hub execution onto the one write entry and deletes their `*_locked` twins. Delivers
`engineering:one-write-entry` for this area; spec [areas.md](../spec/areas.md) and
[write-entry.md](../spec/write-entry.md).

## Change

- Each plain write in the area takes its `transaction` parameter, or a `LockedTransaction` with no default where it is
  correct only under a lock.
- Every caller in the area opens its unit through `IHubTransactions.begin`.
- The area's `*_locked` twins, and the locked-handle reads it used, are deleted.

## Acceptance

- No `*_locked` method of the area remains, and no caller in the area reaches the legacy seam.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
