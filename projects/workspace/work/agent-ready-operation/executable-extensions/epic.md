---
epic: agent-ready-operation/executable-extensions
refinement: outline
status: planned
---

# Run external workspace contributions deliberately

Part of `initiative:agent-ready-operation`; product requirements live in [requirements.md](./requirements.md). Supports
extension activation, lifecycle contributions, diagnostics, and externally supplied capabilities.

## The problem

An engineer needs an extension's hooks and checks without allowing a broken plugin to take over Workspace or hang an
unattended agent. The existing Python callbacks and arbitrary UI objects cannot be a compatibility promise for the Rust
component.

## The outcome

The engineer can inspect contributions without running them, authorize external execution, and distinguish incompatible,
refused, failed, and interrupted contributions. Extension authors can check a declared contract without implicitly
starting services or changing a live environment.

## Depends on

- `epic:parallel-workspaces/component-runtime`
- `epic:parallel-workspaces/workspace-configuration`

## Left for later

`epic:running-environments/service-orchestration` owns provider action semantics and
`epic:parallel-workspaces/environment-lifecycle` owns environment transitions. Dashboard contribution scope belongs to
`epic:agent-ready-operation/workspace-dashboard`; this epic supplies an execution boundary, not a new UI extension
model.

## What this epic is not

- An embedded Python interpreter, Python import compatibility layer, or fallback to the legacy CLI.
- A finalized shared Platform extension API or arbitrary backend flag passthrough.
