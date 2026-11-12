---
name: converge-decisions-and-escalations
depends-on:
  - store-judgments
---

# Converge a chunk's decisions and escalations

Moves the writes of a chunk's decisions and escalations onto the one write entry and deletes their `*_locked` twins.
Delivers `engineering:one-write-entry` for this area; spec
[areas.md](../spec/areas.md#decisions-and-escalations-converge-decisions-and-escalations) and
[write-entry.md](../spec/write-entry.md).

## Change

- Each plain write in the area takes its `transaction` parameter, or a `LockedTransaction` with no default where it is
  correct only under a lock.
- The area's `*_locked` twins, and the locked-handle reads it used, are deleted.
- Lock sites that switch to `IHubTransactions.begin` here: `DecisionService.submit`.
- A cross-area site that calls the area keeps its legacy opener and hands the converted methods `handle.transaction`,
  the transitional form.

## Acceptance

- No `*_locked` method of the area remains, and the lock sites above no longer reach the legacy seam.
- Every remaining caller of the area's writes passes a transaction; none calls a twin.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
