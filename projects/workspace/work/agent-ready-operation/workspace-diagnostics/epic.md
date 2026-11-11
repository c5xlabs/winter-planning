---
epic: agent-ready-operation/workspace-diagnostics
refinement: outline
status: planned
---

# Diagnose a workspace before repairing it

Part of `initiative:agent-ready-operation`; product requirements live in `requirements.md`.

## The problem

An engineer facing a failed agent workflow cannot readily distinguish missing repositories, stale harness guidance,
invalid resource declarations, or a broken service-provider binding. Guessing at repairs risks unrelated work and makes
an unattended agent repeat setup without knowing what was wrong.

## The outcome

The engineer receives named health observations, their severity and source, and usable repair guidance without a
built-in check applying that repair. Optional trusted contributions can add project-specific checks while their
execution and failures remain visible.

## What this epic is not

Content convention checking belongs to `epic:agent-ready-operation/context-quality`. Reconciliation, provisioning, and
service startup remain explicit actions owned by their respective capabilities rather than diagnostic side effects.

## Depends on

- `epic:parallel-workspaces/workspace-configuration`
- `epic:parallel-workspaces/repository-catalog`

Health coverage integrates the facts exposed by `epic:parallel-workspaces/environment-lifecycle`,
`epic:agent-ready-operation/extension-management`, `epic:agent-ready-operation/harness-projection`,
`epic:agent-ready-operation/agent-model-resolution`, and `epic:running-environments/resource-provisioning` and
`epic:running-environments/service-orchestration` as those capabilities arrive; their full delivery is not a
prerequisite for basic workspace diagnostics.
