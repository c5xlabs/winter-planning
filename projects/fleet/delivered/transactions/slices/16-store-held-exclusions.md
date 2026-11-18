---
name: store-held-exclusions
external: github:paul-gross/blizzard#2163
depends-on:
  - tenant-judgments
  - garden-and-work-item-judgments
  - converge-cross-area
---

# Store-held exclusions

Moves the dependency graph's cycle check, the delete and release that share its lock, and the export cursor's pass lock
onto locks the store holds. Delivers `engineering:store-held-exclusions`. Spec [exclusions.md](../spec/exclusions.md).

## Change

- The cycle check, dependency group, delete, and release declare `DEPENDENCIES_LOCK` in place of the per-tenant
  in-process lock (`decision:transactions/cycle-check-lock`); release opens a unit of its own service.
- An additive revision seeds the `EGRESS_CURSOR` hub lock; a pass appends its cursor under it only when the newest
  cursor is still the one it started from, and a reset appends under it. The in-process pass lock is removed.
- The sweep locks stay in process.

## Acceptance

- Neither in-process lock remains.
- Every dependency declare, group, delete, and release declares `DEPENDENCIES_LOCK`; no other chunk write does.
- A pass whose starting cursor a reset moved appends nothing, and its next pass starts from the reset cursor.
- Behavior is unchanged at every route and command, once the store is migrated
  (`engineering:behavior-preserving-landings`); the change note's `## Upgrade` section names the migration.
