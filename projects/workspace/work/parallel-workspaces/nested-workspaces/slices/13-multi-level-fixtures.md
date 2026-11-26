---
name: multi-level-fixtures
depends-on:
  - nested-init-step
  - status-nested-member
  - teardown-preview
  - child-retirement
---

# Multi-level nested workspaces end to end

Exercises the built binary as both the outer and the child CLI over disposable workspaces nested two levels deep, as
`engineering:nested-loss-evidence` asks, proving the flows of [delegation.md](../spec/delegation.md),
[child-state.md](../spec/child-state.md), and [teardown.md](../spec/teardown.md) through real child invocations, where
the earlier slices use a stand-in. Delivers the end-to-end evidence of `product:verified-child-workspace`,
`product:nested-runtime-allocation`, `product:nested-work-inspection`, and `product:nested-teardown-preservation`.

## Change

- Executable tests in `crates/cli/tests/nested.rs`, each building an outer workspace whose nested project is itself a
  workspace with projects cloned from fixture remotes, with the built binary as `winter` in the tree's `bin/`.
- No production code changes.

## Acceptance

- `ws init alpha` initializes the child and clones its projects, and the child's `ws status --json` reports the
  `port_base` its delegated `base_port` gives; once the child holds an env, its `ws list --json` reports the delegated
  `service_prefix`; the outer source checkout of the nested project is not initialized as a workspace.
- The outer `ws status --json` reports the child's env count; with a child env holding a retained branch, a stash, or a
  detached commit present nowhere else, the row's `unpushed` is true and `ws destroy alpha` refuses naming it.
- A child workspace that declares a nested project of its own: `ws status` of the outer reads through both levels, and
  `ws destroy alpha --force` destroys the innermost envs, then the child's, then the outer's.
- A child env whose teardown fails, as with a child worktree locked by `git worktree lock`, keeps the outer env under
  `--force`, and a re-run after `git worktree unlock` completes it.
- A child that binds `service` and whose `winter service down workspace` fails — within this milestone the child's
  `winter` has no `service` command, and once one exists a bound provider with no program on `PATH` fails — keeps the
  outer env, its worktree, directory, exclude block, and index under `--force`, and the result names the child and the
  failed service stop.
- Run with `WINTER_NESTED_CHAIN` holding the outer root, as a misresolved child would be, `ws status` and
  `ws init alpha` refuse every nested project with `OwnRootOnChain` and start no child command, so no sibling nested
  project is reached.
- `cargo dev gate slice` exits 0.
