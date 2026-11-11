---
epic: parallel-workspaces/repository-catalog
refinement: outline
status: planned
---

# Maintain the workspace's repository inventory

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
parallel development, reproducible inputs, and bootstrap through explicit repository identities and roles.

## The problem

Engineers need some repositories multiplied per feature and others shared once. Confusing a role, alias, pin, or path
can direct bulk work at the wrong checkout, while changing the inventory can accidentally be mistaken for deletion.

## The outcome

The engineer can list and edit the declared inventory, understand each repository's role and location, and distinguish
changing a declaration from cloning, branching, updating, or deleting its checkout.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`

## Left for later

Materialization and orphan cleanup belong to `epic:parallel-workspaces/workspace-bootstrap`; environment worktrees to
`epic:parallel-workspaces/environment-lifecycle`; standalone ref resolution and locks to
`epic:parallel-workspaces/reproducible-inputs`. Extension behavior belongs to
`epic:agent-ready-operation/extension-management`, not every repository in the catalog.
