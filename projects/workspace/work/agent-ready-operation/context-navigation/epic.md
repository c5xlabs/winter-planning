---
epic: agent-ready-operation/context-navigation
refinement: outline
status: planned
---

# Read guidance matching the workspace tool

Part of `initiative:agent-ready-operation`; product requirements live in [requirements.md](./requirements.md).

## The problem

Agents currently read copied framework documents and several different extension paths. Once the workspace stops
carrying framework source, they still need to find the right conventions before acting, without loading every guide.

## The outcome

The person or agent starts with a small context entry point and follows the branch their task needs, across built-in
framework guidance, installed extension context, and project-owned context. Framework guidance matches the selected
Workspace component, and essential safety rules reach the agent before it can mutate an environment.

The common eager safety text and context entry hook belong to this epic, with new-workspace placement in
`epic:parallel-workspaces/workspace-bootstrap`; the first milestone does not defer that minimum to later harness
integration.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`

## Left for later

Harness-specific projection, richer delivery, and managed skills/agents belong to
`epic:agent-ready-operation/harness-projection`; extension installation belongs to
`epic:agent-ready-operation/extension-management`. Built-in and project context remain useful without extensions.
