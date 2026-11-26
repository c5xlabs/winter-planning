---
name: status-snapshot
depends-on:
  - target-patterns
  - fan-out
  - worktree-probe
  - ws-fetch
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

- `crates/core/src/status.rs` gains `EnvRow`, `CheckoutRow`, `ProjectRow`, `Snapshot`, `Unseen`, `StatusError`,
  `collect`, and `exit_code`: `collect` does root discovery and the load from the start path, a failure
  `StatusError::Config`; an environment's feature branch from its first non-pinned worktree tracking `origin/`, its
  index from the built `env_index::lookup`, and the first probe failure, in environment then project order, the
  collection error, `StatusError::Collection`.
- A standalone's ahead and behind are counted against `tracking_operand` of `ws-fetch`, and its absence is tested by
  `standalones` of `target-patterns` ([refresh.md](../spec/refresh.md) and
  [target-patterns.md](../spec/target-patterns.md), Domain).
- A missing matched worktree and a missing or failing standalone count in `Unseen`, and so does the `failed_refreshes`
  count `collect` is given; `Unseen` feeds `exit_code` and never the document.
- Nothing is written to any repository or file. `EnvRow`, `CheckoutRow`, `ProjectRow`, `Snapshot`, `Unseen`,
  `StatusError`, `collect`, and `exit_code` are re-exported by name from `lib.rs` for its tests, and
  `StatusCollectionError` on the `termination` line ([status.md](../spec/status.md), Domain); no command calls them yet.

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
- A component test: two non-pinned worktrees of one environment tracking different branches keep their own `upstream`
  per row, and the environment's `feature_branch` comes from the first of them alone.
- A component test: an absent standalone has no row, is warned, and makes `exit_code` 1 when there are no PATTERNS.
- A component test: `collect` leaves every file under the workspace and every ref unchanged.
- A component test: a clean workspace collected with `failed_refreshes` 1 makes `exit_code` 1, with or without PATTERNS,
  and the document is the same as with 0.
- A component test: a start whose configuration does not parse is `StatusError::Config`, not the collection error.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `nested` member and its reads belong to `epic:parallel-workspaces/nested-workspaces`.
