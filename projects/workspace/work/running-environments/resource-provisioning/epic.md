---
epic: running-environments/resource-provisioning
refinement: outline
status: planned
---

# Prepare and clean an environment repeatably

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
runnable environments and tenant handover through declared dependency, resource, data, and cleanup operations.

## The problem

Engineers repeat setup across repositories and cannot tell which resources remain after a failed command. Removing
artifacts is also different from deleting worktrees or shared infrastructure, but the distinction is easy to miss.

## The outcome

The engineer can apply, reset, destroy, or clean declared targets, preview their reach, and receive a truthful account
of completed and remaining work. Base stage execution is usable without a service provider; composed bring-up also
fulfills the charter's safe-to-repeat, one-command working environment promise.

## Depends on

- `epic:running-environments/scope-environment`
- `epic:agent-ready-operation/executable-extensions`

## Left for later

`product:required-service-preconditions` and `product:composed-environment-bringup` integrate with
`epic:running-environments/service-orchestration` when available; neither is an epic-level prerequisite for stage
execution. Composed structural reconcile and destroy integration are shared later with
`epic:parallel-workspaces/environment-lifecycle`, without a reverse dependency from registry identity to provisioning.

## What this epic is not

- A transactional resource manager that can undo arbitrary external commands.
- A rewrite of user-authored setup programs or git-level worktree cleaning.
