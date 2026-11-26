---
name: push-default
depends-on:
  - epic:parallel-workspaces/workspace-bootstrap
---

# The push default read and write

Adds `git::push_default` and `git::set_push_default_upstream` of [push-default.md](../spec/push-default.md): the read of
a checkout's effective `push.default` and the write of `push.default=upstream` to its project's shared configuration.
This is the contract `ws init` calls in `worktree-reconciliation`, and `ws connect` of
`epic:parallel-workspaces/coordinated-git` and `ws checkout` of `epic:parallel-workspaces/safe-history` call after this
epic. Delivers the mechanism of `product:parallel-workspaces/upstream-push-default`. It uses the `GitOperation::Config`
that `epic:parallel-workspaces/workspace-bootstrap` adds.

## Change

- `crates/core/src/git.rs` gains `push_default` and `set_push_default_upstream`, each a `git config` call through the
  program port with the span `git config`; a non-zero exit other than `push_default`'s exit 1 is
  `GitFailure::Repo { Config, repo }`.
- No command calls them yet.

## Acceptance

- Component tests with real git and an isolated `HOME`: `push_default` answers `None` when nothing sets it, and the
  merged value when only the global configuration sets it.
- A component test: `set_push_default_upstream` in a linked worktree writes `$GIT_COMMON_DIR/config`, so `push_default`
  of the source checkout and of a second worktree of the project both answer `upstream`.
- A component test: with `extensions.worktreeConfig` on, the value still lands in the common `config` and the worktree's
  `config.worktree` is unchanged.
- A component test: with `config.lock` present in the common directory, `set_push_default_upstream` is
  `GitFailure::Repo` with `Config` and Git's cause.
- `cargo dev gate slice` exits 0.
