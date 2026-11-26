---
name: ws-pull
depends-on:
  - ws-fetch
  - integration
---

# The `ws pull` command

Adds `refresh::pull`, `PullEvent`, `Synced`, `SyncResult`, and the `ws pull` command of
[refresh.md](../spec/refresh.md), `ws pull`, Events, and its text table: each project's one refresh, then each selected
worktree integrated onto its own upstream through `integrate` of `integration`, pinned worktrees onto
`origin/<main_branch>`, and each standalone that declares a `ref` advanced through `advance` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md) with that
`integrate` as its `IntegrateCheckout`. Delivers the pull bullets of `product:integrate-selected-repositories` and
`product:worktree-upstream-control`, and `engineering:shared-git-effects`, under
`decision:parallel-workspaces/coordinated-git/fetch-freshness` and
`decision:parallel-workspaces/coordinated-git/integration-conflicts`, whose `refresh_failed` and new results are
registered deviations.

## Change

- `crates/core/src/refresh.rs` gains `pull`: a failed refresh is `refresh_failed` and the project's worktrees still
  integrate on local refs; a worktree's upstream is `tracking_operand` of `ws-fetch` with the commit check, so one with
  no upstream, a configured but unfetched one included, is `no_upstream`.
- `crates/core/src/commands/ws/pull.rs` registers the command with `--ff-only`, `--merge`, `--rebase`, and
  `--autostash`, writes the events or their text rows and the diverged hint, and exits 1 unless the run succeeded.
- `crates/core/topics/ws/pull.md`, referenced from the `ws` group topic.

## Acceptance

- Component tests through `InProcess` with fixture remotes: a worktree behind its own upstream fast-forwards; two
  worktrees of one environment tracking different branches each integrate their own; a pinned worktree integrates
  `origin/<main>`.
- Component tests: a diverged worktree is `diverged` and the hint follows; with `--merge` it is `merged`; a dirty one is
  `dirty_blocked`; each makes the run exit 1 but `merged`.
- A component test: with the remote gone, `refresh_failed` names the project, the worktrees still integrate on the refs
  already local, and the command exits 1.
- A component test: a standalone pinned at a tag is `held_pin` and its HEAD does not move; one pinned at a branch is
  `re_pinned` when it fast-forwards and its lock record advances.
- A component test: `--standalone` with an absent standalone warns it, names it in a `target_missing` event under
  `--json`, and starts no git process in it.
- An executable test in `crates/cli/tests` running two `ws pull` processes of the built binary at once over one project
  with two environments: each target ends integrated or `error` with Git's lock cause, never `diverged`.
- `cargo dev gate slice` exits 0.
