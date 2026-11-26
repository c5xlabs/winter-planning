---
name: ws-init-env
depends-on:
  - env-name-rule
  - registry-writes
  - worktree-reconciliation
  - epic:parallel-workspaces/workspace-bootstrap
---

# `ws init <env>` creates and reconciles one environment

Adds `env_create::reconcile_env` and the `TARGET` argument of `ws init` of [init.md](../spec/init.md), Running
`ws init <env>` steps a to f and Fan-out, with the allocation report of [allocation.md](../spec/allocation.md),
Reporting an allocation in `ws init <env>`. The command is the `ws init` that
`epic:parallel-workspaces/workspace-bootstrap` registers without `TARGET`; its row is that of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands.
Delivers `product:environment-worktrees`, the creation half of `product:stable-environment-allocation`, and
`product:environment-recovery` for creation, under
`decision:parallel-workspaces/environment-lifecycle/allocation-policy` and
`decision:parallel-workspaces/partial-execution`; the repository fan-out follows `engineering:bounded-fan-out`.

## Change

- `crates/core/src/env_create.rs` holds `reconcile_env`: allocate before the env directory exists, then the directory,
  the `winter-dir/<env>` self-exclude block, the source check, inference over the existing non-pinned worktrees, and
  `worktree::run` per ready project, at most 4 at once, in reconciliation.md's fan-out.
- The `moved` callback of `state_registry::allocate` emits the `index_moved` action before the write; an allocation
  error fails the target with nothing created; a directory failure keeps the recorded index.
- `bootstrap.rs`'s `Action` gains `IndexMoved`.
- `crates/core/src/commands/ws/init.rs` registers `TARGET`; with it, `ws init` refuses `workspace` as command-surface.md
  gives, then applies `env_name::check`, then reads the registry, each before any git call or write.
- `crates/core/topics/ws/init.md` describes `ws init <env>` and states that its success claims no dependency, resource,
  data, or service readiness.

## Acceptance

- A component test through `InProcess` with fixture source checkouts: `ws init alpha` records an index, writes the
  `/alpha/` exclude block, creates one worktree per project, emits the events of init.md's Flows in order, and exits 0;
  a second run reports each worktree `exists`, writes no exclude file, runs every `cmd` again, and exits 0.
- Component tests: `ws init projects`, `ws init .`, `ws init a/b`, and `ws init .hidden` each exit 1 with nothing
  written and no git process started, as a recording program port shows.
- A component test: with every band slot taken, `ws init beta` exits 1 naming the full band, and neither `<root>/beta`
  nor a registry entry for it exists.
- A component test: an alias recorded off its position reports `index_moved` with `<from> -> <to>` before
  `.winter/state.toml` changes, and the run goes on to create its worktrees.
- A component test: a state file that does not parse ends the command with its `ConfigError`, exit 1, and nothing
  created.
- A component test: one project whose source checkout is missing fails with `SourceCheckoutMissing` and its guidance,
  the other projects' worktrees are created, and the command exits 1.
- A component test: a directory that cannot be created fails the target and leaves the recorded index in place.
- Component tests of inference: one connected sibling upstream is inferred into the new worktree; two disagreeing
  siblings infer nothing; a worktree created in the same run does not vote.
- A unit test over a recording program port: never more than 4 projects in flight, and one project's events keep their
  order; a failing project does not stop another.
- An executable test in `crates/cli/tests` running `ws init alpha` and `ws init beta` of the built binary at once over
  one workspace: both exit 0 and the registry records different indexes.
- `cargo dev gate slice` exits 0.

## Out of scope

- Step g, the `on_env_init` hooks, belongs to `epic:agent-ready-operation/executable-extensions`; the nested step to
  `epic:parallel-workspaces/nested-workspaces`.
- `--all` lands with `ws-init-all`.
