---
name: reference-scenarios
external: github:paul-gross/winter#256
depends-on:
  - ws-init-command
  - prune-command
---

# Recorded scenarios of `ws init` and `ws prune`

Records bare `ws init` and `ws prune` against the reference tool and replays them against the binary, proving the flows
of [reconciliation.md](../spec/reconciliation.md) and [prune.md](../spec/prune.md) end to end under
`decision:functional-parity`; each registered deviation of this epic carries its `[deviation]` decision and expected
result.

## Change

- Scenarios under `scenarios/ws-init/` and `scenarios/ws-prune/`, recorded with `cargo dev record` from the reference
  tool, each with a fixture workspace whose repositories clone fixture remotes; event order across repositories is
  compared with `fan_out`.
- Every fixture standalone declares `extension = false`, and no fixture project holds a `winter-ext.toml` or declares
  `load` or `entry`, and no fixture holds a workspace skills directory, so the reference runs no extension projection or
  skill projection, `AGENTS.winter.md` render, or hook, which this binary leaves to
  `epic:agent-ready-operation/extension-management`; a recording that still shows one is cut down until it does not.
- No production code changes.

## Acceptance

- `ws-init` scenarios: a first run cloning a project and a pinned standalone; a repeat run; a project with a failing
  `cmd`; a missing `url`; excludes appended once; a standalone removed from the configuration, its block stripped; a
  workspace that is a linked worktree.
- `ws-init` deviation scenarios, each with
  `[deviation] decision =
  "decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope"` or
  `decision:parallel-workspaces/workspace-bootstrap/clone-confinement` and its `expected/`: an existing location with a
  different `origin`; a standalone path through a link out of the root.
- `ws-prune` scenarios: a dry run; `--force` removing a clean undeclared clone; a dirty one retained.
- `ws-prune` deviation scenarios under `decision:parallel-workspaces/workspace-bootstrap/prune-retention`: a stale
  standalone directory without `.git` retained; a clone with a local-only commit retained; `--json` alone refused.
- `cargo dev gate full` exits 0.
