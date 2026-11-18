---
name: converge-cross-area
external: github:paul-gross/blizzard#2160
depends-on:
  - converge-chunk-lifecycle
  - converge-chunk-movement
  - converge-chunk-route
  - converge-chunk-dependencies
  - converge-decisions-and-escalations
  - converge-hub-execution
  - converge-record-and-work-refs
---

# Converge the cross-area sites

Switches the lock sites that call the writes of more than one area onto the one write entry, once every area they call
has converged. Delivers `engineering:one-write-entry` for these sites; spec
[areas.md](../spec/areas.md#cross-area-sites-converge-cross-area) and [write-entry.md](../spec/write-entry.md).

## Change

- Each of the eleven cross-area sites opens its unit through `IHubTransactions.begin` in place of its legacy opener.
- The `ILockedChunkRead`-typed helpers in `hub_node.py` take a `LockedTransaction` in place of the handle;
  `_group_locked` and `_escalate_locked` are renamed to drop the marker.
- Each site's reads join its unit, including those that miss it today.

## Acceptance

- No cross-area site reaches the legacy seam; only the graph's callers still do.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
