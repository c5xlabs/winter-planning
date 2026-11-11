---
initiative: parallel-workspaces
refinement: outline
epics:
  - component-runtime
  - workspace-configuration
  - repository-catalog
  - reproducible-inputs
  - workspace-bootstrap
  - environment-lifecycle
  - coordinated-git
  - safe-history
  - stacked-history
  - nested-workspaces
---

# Manage parallel workspaces

Engineers and coding agents need isolated places to change a multi-repository system without losing another task's work.

## The problem

A task spans repository checkouts, branches, tracking settings, and machine-local environment records. Managing those
individually makes setup repetitive and mistakes expensive, especially when work shares Git stores or a feature
environment contains another workspace. The current framework fork also mixes installing Winter with configuring the
engineer's own system.

## The outcome

`epic:parallel-workspaces/component-runtime` supplies an independently callable Workspace component behind the platform
CLI. `epic:parallel-workspaces/workspace-configuration`, `epic:parallel-workspaces/repository-catalog`, and
`epic:parallel-workspaces/reproducible-inputs` make declarations, repository roles, and selected revisions explainable.
`epic:parallel-workspaces/workspace-bootstrap` creates a user-owned workspace without carrying Winter's framework source
and preserves authored instructions while placing the common safety hook.

`epic:parallel-workspaces/environment-lifecycle` gives tasks stable allocations and their own worktrees.
`epic:parallel-workspaces/coordinated-git` respects each repository's tracking when synchronizing or publishing work.
`epic:parallel-workspaces/safe-history` and `epic:parallel-workspaces/stacked-history` make deliberate pointer changes,
deletion, and dependent-branch rewrites recoverable where the operation permits.
`epic:parallel-workspaces/nested-workspaces` extends root verification, allocation isolation, and loss checks to child
workspaces before their stores are removed.

## Left for later

Runtime preparation and service control belong to `initiative:running-environments`; nesting consumes those capabilities
without making base environment identity depend on complete runtime integration. Version-matched guidance comes from
`epic:agent-ready-operation/context-navigation`, while adoption and release evidence belong to
`initiative:rust-adoption`.

## Stance

The selected executable is not the selected target or a sandbox. Successful preflight does not promise transactional
execution: refusals, partial changes, retained work, and recovery remain visible. Shared stores and generated support
files are managed only within their established ownership, and local parallel development does not require Fleet or a
published release.
