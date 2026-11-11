---
epic: agent-ready-operation/harness-projection
refinement: outline
status: planned
---

# Deliver agent capabilities without overwriting local work

Part of `initiative:agent-ready-operation`; product requirements live in [requirements.md](./requirements.md). Supports
supported-harness discovery, workspace initialization, and projection diagnostics.

## The problem

A harness engineer needs the same skills, agents, and context on Claude Code, Codex, and OpenCode, but each discovers
different files. Generated copies can go stale, and prefix-based cleanup cannot distinguish a user file from Winter's
own output.

## The outcome

The engineer can reconcile supported harness artifacts from canonical sources, inspect their condition, and recover from
invalid sources or interrupted writes without corrupting sources or unrelated local work. Unsupported permissions and
identity differences remain visible.

## Depends on

- `epic:agent-ready-operation/extension-management`
- `epic:agent-ready-operation/agent-model-resolution`
- `epic:agent-ready-operation/context-navigation`

## Left for later

`epic:parallel-workspaces/workspace-bootstrap` owns the overall initialization entry point;
`epic:agent-ready-operation/workspace-diagnostics` owns aggregate health output.
`epic:rust-adoption/workspace-migration` owns migration authorization and `epic:rust-adoption/parity-validation` owns
adoption evidence across harness versions.

## What this epic is not

- A guarantee that every future harness version supports today's file layout or native permissions.
- Rewriting reusable procedure bodies into harness-specific workflows or launching agent sessions.
