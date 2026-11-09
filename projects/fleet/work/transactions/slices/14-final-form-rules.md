---
name: final-form-rules
depends-on:
  - announce-after-commit
  - converge-graph
  - adapters-stay-apart
  - store-held-exclusions
---

# Final-form rules

Removes the legacy clauses and Known debt from the write rules and lands a gate for each departure they catch at review.
Delivers `engineering:final-form-rules` and `product:one-way-to-write`.

## Change

- `bzh:transaction-participation`, `bzh:store-exclusive-write`, and `bzh:store-refuses-stale` lose their legacy clauses
  and Known debt.
- Gates fail on a `*_locked` name or locked-handle type under `hub/`, a store write that reads a status-family fact to
  refuse, and a broker call inside a store write.

## Acceptance

- Each gate fails on a planted instance of its departure and passes on the tree.
