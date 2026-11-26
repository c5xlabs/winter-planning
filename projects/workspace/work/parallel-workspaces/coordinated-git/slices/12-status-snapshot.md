---
name: status-snapshot
depends-on:
  - target-patterns
  - fan-out
  - worktree-probe
  - epic:parallel-workspaces/workspace-bootstrap
---

# The status snapshot and its exit code

Adds `status::collect`, `Snapshot`, and `exit_code` of [status.md](../spec/status.md), Environments, Projects,
standalones, workspace, and Exit code: every environment present probed by `worktree-probe` at most `STATUS_PROBES` at
once and filtered by PATTERNS, the source checkouts and standalones with their local work, and the workspace level from
`drift::detect`, `scan` of [prune.md](../../workspace-bootstrap/spec/prune.md), which
`epic:parallel-workspaces/workspace-bootstrap` builds, and `pins::snapshot`. Delivers the read-only bullets of
`product:inspect-workspace-git` and the per-worktree tracking of `product:worktree-upstream-control`, under
`decision:parallel-workspaces/target-selection`, whose missing-worktree exit is a registered deviation.

## Change

- `crates/core/src/status.rs` gains `EnvRow`, `CheckoutRow`, `ProjectRow`, `Snapshot`, `Unseen`, `collect`, and
  `exit_code`: an environment's feature branch from its first non-pinned worktree tracking `origin/`, its index from the
  built `env_index::lookup`, and the first probe failure, in environment then project order, the collection error.
- A missing matched worktree, a missing or failing standalone, and a failed refresh count in `Unseen`, which feeds
  `exit_code` and never the document.
- Nothing is written to any repository or file; no command calls it yet.

## Acceptance

- Component tests with real git: a two-environment workspace gives one row per project per environment, the projects
  with drift or local work, each standalone, the workspace's drift lists, orphans, and pins, and the resolved dashboard
  layout.
- A component test: PATTERNS matching one environment's rows drop the other environment; matching nothing is the
  collection error naming the patterns.
- A component test: an absent matched worktree has no row, is warned, and makes `exit_code` 1.
- A component test: a failing standalone probe is a warning and no row, and the other standalones are present.
- A component test: a worktree probe failure in one environment is the collection error after that environment's other
  probes ran.
- A component test: two environments tracking different upstreams keep their own `upstream` per row.
- A component test: `collect` leaves every file under the workspace and every ref unchanged.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `nested` member and its reads belong to `epic:parallel-workspaces/nested-workspaces`.
