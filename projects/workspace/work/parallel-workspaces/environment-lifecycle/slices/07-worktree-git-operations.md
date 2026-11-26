---
name: worktree-git-operations
depends-on:
  - module-scaffold
---

# The worktree and tracking Git operations

Adds the Git operations of [init.md](../spec/init.md), Git operations: `branch_exists`, `worktree_add`,
`worktree_add_new`, `worktree_list`, and `common_dir` in `git.rs`, and the tracking reads and write `head_branch`,
`setting`, `tracking`, and `set_upstream` in `env_create/worktree.rs`, which `ws init`, `ws destroy`, and `ws list`
share. Serves `product:environment-worktrees` and the store and registration checks of
`engineering:worktree-effect-confinement`. Every call goes through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md) with its
`git <subcommand>` span.

## Change

- `crates/core/src/git.rs` gains the five operations, each failing as `GitFailure` with its `GitOperation`.
- `crates/core/src/env_create/worktree.rs` gains `head_branch`, `setting`, `tracking`, and `set_upstream`, `pub(crate)`
  so `ws list` reads them; `set_upstream` writes `branch.<head>.remote` and `branch.<head>.merge` directly, so a
  never-pushed branch can be the upstream.
- No command calls them yet.

## Acceptance

- Component tests with real git: `branch_exists` answers true and false by its exit code, and another exit is a
  `GitFailure`; `worktree_add_new` creates the branch from the project's main branch with no `branch.<b>.remote` set;
  `worktree_add` checks out an existing local branch; `worktree_list` answers every `worktree <path>` line, a path
  holding a space included; `common_dir` answers the same absolute, resolved path for a source checkout and its
  worktree.
- Component tests: `head_branch` answers `None` for a detached HEAD; `tracking` answers `None` without both keys and
  `<remote>/<branch>` with them; `set_upstream` makes `tracking` answer the upstream for a branch that origin does not
  have; an upstream with an empty branch is `BadUpstream`, and a detached HEAD is `HeadDetached`.
- `cargo dev gate slice` exits 0.
