---
epic: agent-ready-operation/context-navigation
refinement: outline
status: in-progress
external: github:paul-gross/winter#236
---

# Read guidance matching the workspace tool

Part of `initiative:agent-ready-operation`; product requirements live in [requirements.md](./requirements.md).

## The problem

Agents currently read Winter's framework guidance from documents copied into each workspace. Once the workspace stops
carrying framework source, they still need to find the right conventions before acting, without loading every guide.

## The outcome

The person or agent starts with a small context entry point in the Workspace component and asks for the topic their task
needs. The guidance matches the installed Workspace version, and essential safety rules reach the agent before it can
mutate an environment.

The context covers Winter's own framework guidance only. Extensions and projects are repositories, so their guidance
stays in their own Markdown files, read by path. The common eager safety text belongs to this epic; placing it in a new
or reconciled workspace belongs to `epic:parallel-workspaces/workspace-bootstrap`, and the first milestone does not
defer that minimum to later harness integration.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`

## Left for later

Harness-specific projection, richer delivery, and managed skills/agents belong to
`epic:agent-ready-operation/harness-projection`; extension installation and extension context entry points belong to
`epic:agent-ready-operation/extension-management`. The setup topics themselves are added by
`epic:parallel-workspaces/workspace-bootstrap` and `epic:running-environments/service-orchestration`. Explaining the
effective configuration stays open under
`decision:parallel-workspaces/workspace-configuration/configuration-inspection`; this epic adds no inspection topic or
command.
