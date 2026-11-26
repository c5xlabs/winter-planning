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
with its sibling's ports, or lose entire Git stores when the outer task is removed, leaving the child's services running
from a directory that no longer exists.

## The outcome

The engineer can initialize, inspect, and retire a declared nested workspace with verified child identity, isolated
allocations, deliberately inherited local settings, and recursive protection for work that would disappear. Retiring the
outer environment retires each child first, its services and provisioned resources included.

## Depends on

- `epic:parallel-workspaces/environment-lifecycle`

## Left for later

Nothing runtime-shaped is left to another epic. The child's runtime is retired by invoking the child's own CLI: its
`ws destroy` runs the child's provision teardown and stops its env services, and its `service down workspace` stops its
workspace services, so this epic needs neither the service-orchestration nor the provisioning capability of the outer
workspace. Provider behaviour inside the child stays with the child's own providers.
