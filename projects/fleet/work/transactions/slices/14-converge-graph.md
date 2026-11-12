---
name: converge-graph
external: github:paul-gross/blizzard#2161
depends-on:
  - config-judgments
  - converge-cross-area
---

# Converge the graph

Moves the writes of the graph onto the one write entry, deletes their `*_locked` twin, and removes the legacy seam with
its last caller. Delivers `engineering:one-write-entry`; spec
[areas.md](../spec/areas.md#the-graph-and-the-seams-removal-converge-graph) and
[write-entry.md](../spec/write-entry.md).

## Change

- `IWriteGraphRepository.mint_locked` folds into `mint`, taking a `LockedTransaction`; `get_enabled_by_name` and
  `any_minted` take `transaction`; the `GRAPH_NAME` key builder moves to the domain.
- The default-graph mint opens its unit through `IHubTransactions.begin`, and `GraphStore` stops opening its own write
  for `locked_name`.
- The legacy seam is deleted: everything write-entry.md §Removed lists, including `effective_locked` and
  `_retire_locked`, renamed to drop the marker.

## Acceptance

- No `*_locked` method or helper and no locked-handle type remains under `hub/`.
- Every write that must win exactly once takes its locks through one `begin` declaration, but the declared exceptions
  (write-entry.md §Opening a write).
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
