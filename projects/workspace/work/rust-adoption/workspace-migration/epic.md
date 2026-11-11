---
epic: rust-adoption/workspace-migration
refinement: outline
status: planned
---

# Bring an existing workspace forward

Part of `initiative:rust-adoption`; product requirements live in [requirements.md](./requirements.md).

## The problem

An engineer's workspace currently carries framework source, inherited documentation, local overlays, and generated agent
files alongside their own configuration and context. Replacing that tree indiscriminately would destroy the
customization the workspace exists to preserve.

## The outcome

The engineer can inspect an existing workspace, learn which parts need conversion, and deliberately adopt the Rust
component while preserving repositories, feature work, local settings, and project-owned context. Unsupported inputs are
named before conversion rather than silently ignored.

## Depends on

- `epic:parallel-workspaces/workspace-bootstrap`
- `epic:agent-ready-operation/context-navigation`
- `epic:parallel-workspaces/environment-lifecycle`
- `epic:agent-ready-operation/harness-projection`

## What this epic is not

- A Python command compatibility shim or automatic upstream-history rewrite.
- Permission to remove a user's framework fork or private customization without inspection.
