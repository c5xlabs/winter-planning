---
epic: parallel-workspaces/repository-catalog
refinement: final
status: in-progress
external: github:paul-gross/winter#196
---

# Maintain the workspace's repository inventory

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
parallel development, reproducible inputs, and bootstrap through explicit repository identities and roles.

## The problem

Engineers need some repositories multiplied per feature and others shared once. Confusing a role, alias, pin, or path
can direct bulk work at the wrong checkout, while changing the inventory can accidentally be mistaken for deletion.

## The outcome

The engineer can list and edit the declared inventory, understand each repository's role and location, find every
checkout location that exists, and distinguish changing a declaration from cloning, branching, updating, or deleting its
checkout.

## Depends on

- `epic:parallel-workspaces/component-runtime`
- `epic:parallel-workspaces/workspace-configuration`
- `epic:rust-adoption/parity-validation`

## Left for later

Cloning a declared repository, confining what a clone may write to, and removing orphaned checkouts belong to
`epic:parallel-workspaces/workspace-bootstrap`, and so does refusing overlapping, reserved, or symlinked locations at
the point of cloning, if that is wanted. Showing each repository's main branch before bulk Git work, and how a pinned
project takes part in feature branches, belong to `epic:parallel-workspaces/coordinated-git` and
`epic:parallel-workspaces/environment-lifecycle`; environment worktrees and which environments exist belong to
`epic:parallel-workspaces/environment-lifecycle`; standalone ref resolution and locks to
`epic:parallel-workspaces/reproducible-inputs`. Extension behaviour belongs to
`epic:agent-ready-operation/extension-management`, not every repository in the catalog.

Beyond the reference tool: warning about missing or undeclared standalones, as the drift warning does for projects,
belongs to `epic:agent-ready-operation/workspace-diagnostics`; telling a feature worktree location that is not a Git
repository from a clean one belongs to `epic:parallel-workspaces/coordinated-git`.

## What this epic is not

- A report of which configuration layer an edit changed: the engineer chose it with `--local`, and nothing later is
  planned to report it.
- A source normaliser: `…/api` and `…/api.git` stay two sources, and no later work is planned to merge them.
