---
epic: rust-adoption/parity-validation
refinement: outline
status: in-progress
---

# Judge the replacement against real work

Part of `initiative:rust-adoption`. Supports the other capability epics with the method each uses to prove its commands
against the existing Workspace; the account of the whole replacement's coverage belongs to
`epic:rust-adoption/workspace-release`.

## The problem

An engineer cannot trust a replacement that demonstrates only happy paths. The current tool's refusal rules, nested
operations, provider failures, and preservation of hand-authored files matter as much as successful repository setup.

## The outcome

Maintainers can record the existing Workspace's behaviour in an isolated fixture workspace and replay the replacement
against that record, so each command is shown to keep the old behaviour, its refusals included, or to differ only where
a decision says it does, before it is trusted with live feature work.

## Depends on

- `epic:parallel-workspaces/component-runtime`

## What this epic is not

- A promise to reproduce Python internals or every old JSON field regardless of the chosen public contracts.
- A gate that prevents other epics from developing their own tests or shipping usable development builds.
