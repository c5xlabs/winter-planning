---
epic: parallel-workspaces/environment-lifecycle
refinement: outline
status: planned
---

# Give each task a safe feature environment

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
coordinated Git, nested workspaces, and isolated runtime through durable environment identity and worktrees.

## The problem

Engineers and agents need several tasks to coexist without editing source checkouts or sharing environment allocations.
Partial creation and unsafe teardown can strand worktrees, forget reservations, or remove a task's only copy of work.

## The outcome

The engineer can create and reconcile environment worktrees, inspect their identity and allocations, and retire them
with visible safety checks and recovery information. Structural creation remains distinct from runtime readiness.

## Depends on

- `epic:parallel-workspaces/repository-catalog`

## Left for later

Whole-workspace bootstrap remains `epic:parallel-workspaces/workspace-bootstrap`. Runtime variable rendering belongs to
`epic:running-environments/scope-environment`; provision and service teardown integration belongs to
`epic:running-environments/resource-provisioning` and `epic:running-environments/service-orchestration`. Those
integrations can land after structural lifecycle without making the capabilities depend on one another in a cycle.
