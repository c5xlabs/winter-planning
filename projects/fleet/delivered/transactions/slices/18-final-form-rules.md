---
name: final-form-rules
external: github:paul-gross/blizzard#2165
depends-on:
  - converge-graph
  - adapters-stay-apart
  - store-held-exclusions
  - postgres-proof
---

# Final-form rules

Removes the legacy clauses and Known debt from the write rules and lands a gate for each store-side departure they catch
at review. Delivers `engineering:final-form-rules` and `product:one-way-to-write`. Spec [gates.md](../spec/gates.md).

## Change

- In blizzard-context, `bzh:transaction-participation`, `bzh:store-exclusive-write`, and `bzh:store-refuses-stale` lose
  their legacy clauses and Known debt (`architecture/system-shape/exclusive-writes.md`,
  `architecture/repository-access.md`).
- In blizzard, ast-grep gates fail on a store write that reads a status-family fact to refuse, and on a broker call
  inside a store write.

## Acceptance

- Each gate fails on a planted instance of its departure and passes on the tree.
- No write rule carries a legacy clause or a Known-debt entry this epic resolves.
