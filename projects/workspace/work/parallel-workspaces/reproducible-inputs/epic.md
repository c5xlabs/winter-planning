---
epic: parallel-workspaces/reproducible-inputs
refinement: outline
status: planned
---

# Reproduce and compare the workspace's inputs

Part of `initiative:parallel-workspaces`; product requirements live in [requirements.md](./requirements.md). Supports
adoption and repeatable agent work through intentional standalone resolutions and workspace identity.

## The problem

Engineers cannot reproduce a workspace when auxiliary inputs move unnoticed, and cannot compare two copies by commit
history alone when local tracked edits differ. A pin that rewrites local work is worse than no reproducibility at all.

## The outcome

The engineer can resolve standalone branch, tag, or commit references, review intentional updates, freeze an existing
checkout, and compare workspace definitions without altering their working state.

## Depends on

- `epic:parallel-workspaces/repository-catalog`

## Left for later

Project main-branch pinning remains catalog policy. Installed Workspace component version selection belongs to Platform;
legacy-lock conversion belongs to `epic:rust-adoption/workspace-migration` under `decision:configuration-transition`.
