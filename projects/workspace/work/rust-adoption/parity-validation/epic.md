---
epic: rust-adoption/parity-validation
refinement: outline
status: planned
---

# Judge the replacement against real work

Part of `initiative:rust-adoption`; product requirements live in [requirements.md](./requirements.md). Supports the
other capability epics by making the replacement's coverage inspectable.

## The problem

An engineer cannot trust a replacement that demonstrates only happy paths. The current tool's refusal rules, nested
operations, provider failures, and preservation of hand-authored files matter as much as successful repository setup.

## The outcome

The engineer can see which existing capabilities the replacement covers, which differ deliberately, and which still
block adoption. Maintainers can exercise the replacement against isolated representative workspaces before risking live
feature work.

## Depends on

- `epic:parallel-workspaces/component-runtime`

## What this epic is not

- A promise to reproduce Python internals or every old JSON field regardless of the chosen public contracts.
- A gate that prevents other epics from developing their own tests or shipping usable development builds.
