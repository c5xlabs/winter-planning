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
- `epic:running-environments/scope-environment`
- `epic:running-environments/resource-provisioning`
- `epic:running-environments/service-orchestration`

## Left for later

Child lifecycle integration can be staged after structural environment support and the runtime epics; none must depend
on nested workspaces to operate independently. Provider implementations remain owned by
`epic:running-environments/provider-tmux`, `epic:running-environments/provider-docker`, and
`epic:running-environments/provider-process-compose`.
