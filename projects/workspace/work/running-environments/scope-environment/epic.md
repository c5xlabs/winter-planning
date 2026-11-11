---
epic: running-environments/scope-environment
refinement: outline
status: planned
---

# Resolve the right environment before running work

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
provisioning, service orchestration, and external execution with one inspectable scope environment.

## The problem

Engineers cannot safely run several environments when a command reads a neighbor's ports or silently launches without
the values its declaration requires. Secrets lookups also must not turn routine inspection into hidden execution.

## The outcome

The engineer can inspect a scope's runtime values, deliberately resolve external values, and trust launching and
reporting operations to apply the same declared scope and precedence rules without leaking values across environments.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
- `epic:parallel-workspaces/environment-lifecycle`

## What this epic is not

- A second environment registry, resource allocator, or structural lifecycle.
- A bundled secrets manager or a rewrite of user-owned value-producing programs.
