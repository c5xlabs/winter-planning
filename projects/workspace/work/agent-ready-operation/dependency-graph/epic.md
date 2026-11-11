---
epic: agent-ready-operation/dependency-graph
refinement: outline
status: planned
---

# Inspect declared module dependencies

Part of `initiative:agent-ready-operation`; product requirements live in `requirements.md`.

## The problem

An extension author cannot tell whether a reference will remain usable outside their development workspace without
knowing which modules are installed and which dependencies were declared. Rebuilding that picture separately in each
check makes disagreement and missing modules difficult to spot.

## The outcome

People and tools can inspect the same declared extension-module dependency picture, see its source and omissions, and
use it when checking whether content can travel independently. It does not map arbitrary context reachability.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
- `epic:parallel-workspaces/repository-catalog`
- `epic:agent-ready-operation/extension-management` for installed extension declarations, not executable plugin loading.
