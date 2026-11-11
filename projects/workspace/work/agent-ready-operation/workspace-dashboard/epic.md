---
epic: agent-ready-operation/workspace-dashboard
refinement: outline
status: planned
---

# Inspect the workspace without losing your place

Part of `initiative:agent-ready-operation`; product requirements live in `requirements.md`.

## The problem

An engineer comparing work across environments has to remember which checkout is dirty, which remote each worktree
tracks, and which service or agent configuration explains a failure. Repeated terminal queries lose that context, while
a dashboard that goes blank or acts on a stale selection makes the shortcut unsafe.

## The outcome

The engineer can navigate a live workspace overview, inspect the selected repository and agent settings, and understand
failures without rerunning them. Deliberate actions use the safety rules of non-interactive operations, and extension
contributions remain useful without importing Python UI code.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
- `epic:parallel-workspaces/repository-catalog`

Environment views integrate `epic:parallel-workspaces/environment-lifecycle` and
`epic:parallel-workspaces/coordinated-git`; service observations consume
`epic:running-environments/service-orchestration`. Agent views and reconciliation integrate
`epic:agent-ready-operation/agent-model-resolution`, `epic:agent-ready-operation/harness-projection`, and
`epic:parallel-workspaces/workspace-bootstrap`, while external actions use
`epic:agent-ready-operation/executable-extensions`. These are capability-level integrations, not a requirement to finish
all lifecycle, provider, diagnostic, and quality work before a repository overview can be delivered.
