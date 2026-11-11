---
epic: parallel-workspaces/workspace-configuration
refinement: outline
status: planned
---

# Know which workspace and configuration will be used

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
the other parity epics through one identifiable workspace and explainable effective configuration.

## The problem

Engineers and agents can invoke a tool from a worktree, nested workspace, or unrelated directory and mistake which
configuration it will use. Local overrides and invalid declarations can silently change the operation they intended.

## The outcome

The caller can identify the selected workspace, understand the configuration that governs an operation, and correct
invalid or incompatible input before it causes effects. Workspace discovery does not depend on carrying tool source.

## Depends on

- `epic:parallel-workspaces/component-runtime`

## Left for later

Repository declarations belong to `epic:parallel-workspaces/repository-catalog`; environment allocations belong to
`epic:parallel-workspaces/environment-lifecycle`. Existing-workspace conversion belongs to
`epic:rust-adoption/workspace-migration` under `decision:configuration-transition`.
