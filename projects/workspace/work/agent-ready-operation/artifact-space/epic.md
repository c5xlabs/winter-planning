---
epic: agent-ready-operation/artifact-space
refinement: outline
status: planned
---

# Give generated work a configurable home

Part of `initiative:agent-ready-operation`; product requirements live in `requirements.md`.

## The problem

An engineer moving between coding harnesses loses generated reviews, scores, or workflow notes when each tool assumes
its own hidden output directory. Hardcoded paths also make it difficult to separate generated work from deliverables.

## The outcome

People and external tools can ask where an artifact kind belongs without creating it, and can move the root or an
individual kind through workspace configuration. This capability can land independently of graph, diagnostics, and TUI.

## What this epic is not

Resolving a directory does not declare ownership of its contents or grant permission to delete them. Declared clean
actions and their safety belong to `epic:running-environments/resource-provisioning`.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
