---
epic: running-environments/nested-runtime
refinement: outline
status: planned
---

# Run and retire a nested workspace's runtime

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
engineers running a whole workspace definition inside an isolated feature environment.

## The problem

Retiring an outer environment can remove a child workspace's stores while its services still run or its provisioned
resources remain. A child's runtime values can also drift from the allocation its parent delegated, so siblings collide
despite a correct structural delegation.

## The outcome

The engineer can retire an outer environment knowing each child's services were stopped and its provisioned resources
torn down first, and can trust that a child's runtime values agree with what the parent delegated.

## Depends on

- `epic:parallel-workspaces/nested-workspaces`
- `epic:running-environments/scope-environment`
- `epic:running-environments/resource-provisioning`
- `epic:running-environments/service-orchestration`

## Left for later

Child identity, allocation delegation, local inheritance, the recursive loss audit, and structural teardown belong to
`epic:parallel-workspaces/nested-workspaces`. Provider implementations remain owned by
`epic:running-environments/provider-tmux`, `epic:running-environments/provider-docker`, and
`epic:running-environments/provider-process-compose`.
