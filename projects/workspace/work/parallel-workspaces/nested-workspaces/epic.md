---
epic: parallel-workspaces/nested-workspaces
refinement: outline
status: planned
---

# Manage a workspace held inside another task

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
engineers developing or exercising whole workspace definitions inside isolated feature environments.

## The problem

A nested checkout can resolve the enclosing workspace by mistake, inherit its credentials and runtime settings, collide
with its sibling's ports, or lose entire Git stores when the outer task is removed.

## The outcome

The engineer can initialize, inspect, and retire a declared nested workspace with verified child identity, isolated
allocations, deliberately inherited local settings, and recursive protection for work that would disappear.

## Depends on

- `epic:parallel-workspaces/environment-lifecycle`

## Left for later

This epic is structural: it needs no runtime capability, and teardown here retires child environments and their stores
only. Stopping a child's services, running its provision teardown, and checking its runtime values against the delegated
allocation belong to `epic:running-environments/nested-runtime`, which builds on this epic and the runtime epics; none
of them depends on nested workspaces to operate independently.
