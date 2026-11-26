---
name: safety-probes
depends-on:
  - module-scaffold
  - epic:parallel-workspaces/coordinated-git
---

# The work-preservation probes

Adds `history::has_ref`, `safety_ref`, `unpushed`, `active_branch`, `commit`, and `Branch` of
[safety-probes.md](../spec/safety-probes.md): the read-only checks `ws checkout` and `ws reset` run before a pointer
moves, over the `Selected` worktree of `epic:parallel-workspaces/coordinated-git`. Unpushed commits are measured from
the moving ref against its configured upstream when that ref exists locally, else against `origin/<main_branch>`, never
against the destination. Delivers `engineering:moved-ref-preservation` and the failed-check bullet of
`product:adopt-feature-history`.

## Change

- `crates/core/src/history.rs` gains the five probes and `Branch`; every git call goes through the program port with its
  `git <subcommand>` span and fails as the repo error with the operation safety-probes.md's table gives.
- The dirty probe is `git::dirty` as built, which the preflights call directly; `history.rs` holds no `dirty` of its
  own. `safety_ref` of `Branch::Named(env)` reads that branch's own upstream, so checkout can measure an env branch HEAD
  is not on.
- A non-zero exit of `git rev-list --count` is the repo error of the call, never a count of zero.
- The five probes and `Branch` are re-exported by name from `lib.rs` for its tests
  ([safety-probes.md](../spec/safety-probes.md), Domain); no command calls them yet.

## Acceptance

- Component tests with real git: `safety_ref(Head)` answers the configured upstream when its ref is fetched, and
  `origin/<main>` when the branch has no upstream, when its upstream is configured but not fetched, and when HEAD is
  detached; `safety_ref(Named(env))` answers `env`'s own upstream while HEAD is on another branch.
- Component tests: `unpushed` counts the commits on the moving ref and not on the safety ref; with no `origin/<main>` it
  is `GitFailure::Repo` with `AbandonmentProbe`, not 0.
- Component tests: `active_branch` answers the branch name, also for an unborn HEAD, and `None` for a detached one;
  `commit` answers the sha of `HEAD` and `None` for an unborn one; `has_ref` is true for a tag, a remote-tracking ref,
  and a commit id, and false for an absent ref.
- `cargo dev gate slice` exits 0.
