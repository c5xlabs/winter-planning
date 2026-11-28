---
name: prune-command
external: github:paul-gross/winter#254
depends-on:
  - workspace-exclude
  - port-extensions
  - module-scaffold
  - ws-init-command
  - epic:agent-ready-operation/context-navigation
code:
  - winter-workspace/crates/core/src/termination.rs::Termination
---

# `ws prune` removes only material that cannot hold work

Adds `prune::scan`, `prune::remove`, `prune::reaggregate`, and the `ws prune [--dry-run] [--force] [--json]` command of
[prune.md](../spec/prune.md), with the `ws/prune` topic of [guidance.md](../spec/guidance.md). Builds the confirmation
of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Confirmation, in
`crates/core/src/confirm.rs`, which no earlier slice builds; the commands of later epics that prompt, `ws clean` and
`ws destroy`, call it; and adds `Termination::Aborted { interrupted }` of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
which arrives with the first command that prompts. Delivers `product:prune-managed-material` and
`engineering:prune-ownership-evidence` under `decision:parallel-workspaces/workspace-bootstrap/prune-retention`.

## Change

- `crates/core/src/prune.rs` holds the candidate scan of the four kinds from `projects/`, the exclude file read through
  `locate`, and the vendor directories; the retention checks in their order, with `git rev-list` and
  `git rev-parse
  --verify --quiet refs/stash` added to `git.rs`; the recheck and removal through `WriteFiles::remove`;
  and `reaggregate` through `finalize` with the retained blocks kept.
- `crates/core/src/commands/ws/prune.rs` registers the command, refuses `--json` without `--force` or `--dry-run` with
  `BootstrapError::JsonNeedsForce`, writes the preview with its evidence, prompts unless `--force` or `--dry-run`, and
  writes the per-orphan lines or the JSON document; the reaggregation line comes from `bootstrap::text`.
- `crates/core/src/termination.rs` gains `Termination::Aborted { interrupted }`, exit 1, with its diagnostic.
- `crates/core/topics/ws/prune.md`, referencing `ws/init`, and referenced from the `ws` group topic; `ws/init` gains its
  reference to `ws/prune`, now that both exist.
- `context/verification/matrix.md` gains `workspace:manual-ws-prune`.

## Acceptance

- Component tests with real git: an undeclared clean clone under `projects/` is removed after a "y" answer and kept
  after "n" (exit 1); a dirty one, one with linked worktrees, one with a local-only commit, one with a stash, and a
  directory without `.git` are each retained with their note, under `--force` too.
- A component test: a standalone directory named by a stale block and holding no `.git` is retained, and its block
  survives the reaggregation that a removal elsewhere triggers.
- A component test: a block line resolving outside the root is retained as `OutsideRoot`.
- A component test: with the root not a git repository, a removal writes no exclude file and `--json --force` reports
  `excludes_updated: true`; removing an undeclared project clone alone from a git root reports `excludes_updated: true`.
- A unit test of `remove` over `MemoryFiles`: a candidate whose path, or whose parent, is a symbolic link where the scan
  saw a directory is `Skipped` "changed since the scan", and the link's target is untouched.
- Component tests: `--json` alone exits 1 and removes nothing; `--json --dry-run` prints `would_remove` and `skipped`
  actions; `--json --force` prints the reaggregation events, then the document with `excludes_updated`.
- A component test: `--dry-run` runs no `git config` write and removes nothing.
- A component test: a broken link under `.claude/skills` is removed; an agent copy named by a stale block is removed.
- A unit test: `Termination::Aborted { interrupted: false }` ends with exit 1 and the notice that the run was aborted; a
  declined prompt removes nothing.
- `cargo dev gate slice` exits 0.
