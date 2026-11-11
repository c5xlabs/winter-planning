---
epic: agent-ready-operation/agent-model-resolution
refinement: outline
status: planned
---

# Explain the model selected for each agent

Part of `initiative:agent-ready-operation`; product requirements live in [requirements.md](./requirements.md). Supports
harness projection and agent configuration inspection.

## The problem

A harness engineer can change a tier or personal override and still be unsure which model an installed agent will use.
Native reasoning settings and permission controls differ across harnesses, so apparently portable settings can conceal
different behavior.

## The outcome

The engineer can configure canonical agents and inspect each harness's resolved model, effort, and winning source before
installing it. Invalid mappings and unsupported permission intent are visible rather than silently converted into an
unrestricted agent.

## Depends on

- `epic:agent-ready-operation/extension-management`
- `epic:parallel-workspaces/workspace-configuration`

## Left for later

`epic:agent-ready-operation/harness-projection` owns installed artifacts and their drift;
`epic:agent-ready-operation/workspace-diagnostics` owns the health-check presentation. Dashboard presentation belongs to
`epic:agent-ready-operation/workspace-dashboard`.

## What this epic is not

- A live model marketplace, vendor availability guarantee, or automatic ranking of vendor capabilities.
- An agent launcher or Fleet execution-placement policy.
