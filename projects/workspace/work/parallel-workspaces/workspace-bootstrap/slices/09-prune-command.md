---
name: prune-command
depends-on:
  - workspace-exclude
  - port-extensions
  - module-scaffold
  - ws-init-command
  - epic:agent-ready-operation/context-navigation
---

# `ws prune` removes only material that cannot hold work

Adds `prune::scan`, `prune::remove`, `prune::reaggregate`, and the `ws prune [--dry-run] [--force] [--json]` command of
[prune.md](../spec/prune.md), with the `ws/prune` topic of [guidance.md](../spec/guidance.md). Builds the confirmation
of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Confirmation, in
`crates/core/src/confirm.rs`, which no earlier slice builds; the commands of later epics that prompt, `ws clean` and
`ws destroy`, call it. Delivers `product:prune-managed-material` and
`engineering:prune-ownership-evidence` under `decision:parallel-workspaces/workspace-bootstrap/prune-retention`.

## Change

- `crates/core/src/prune.rs` holds the candidate scan of the four kinds from `projects/`, the exclude file read through
  `locate`, and the vendor directories; the retention checks in their order, with `git rev-list` and `git rev-parse
  --verify --quiet refs/stash` added to `git.rs`; the recheck and removal through `WriteFiles::remove`; and
  `reaggregate` through `finalize` with the retained blocks kept.
- `crates/core/src/commands/ws/prune.rs` registers the command, refuses `--json` without `--force` or `--dry-run` with
  `BootstrapError::JsonNeedsForce`, writes the preview with its evidence, prompts unless `--force` or `--dry-run`, and
  writes the per-orphan lines or the JSON document.
- `crates/core/topics/ws/prune.md`, referencing `ws/init`, and referenced from the `ws` group topic; `ws/init`
  gains its reference to `ws/prune`, as guidance.md, lands, gives.
- `context/verification/matrix.md` gains `workspace:manual-ws-prune`.

## Acceptance

- Component tests with real git: an undeclared clean clone under `projects/` is removed after a "y" answer and kept
  after "n" (exit 1); a dirty one, one with linked worktrees, one with a local-only commit, one with a stash, and a
  directory without `.git` are each retained with their note, under `--force` too.
- A component test: a standalone directory named by a stale block and holding no `.git` is retained, and its block
  survives the reaggregation that a removal elsewhere triggers.
- A component test: a block line resolving outside the root is retained as `OutsideRoot`; a symbolic link under
  `projects/` to a clean clone is retained as `Link`.
- A component test: with the root not a git repository, a removal elsewhere runs no reaggregation, every exclude block
  is left as it was, and `--json --force` reports `excludes_updated: false`; removing an undeclared project clone alone
  from a git root reports `excludes_updated: true`.
- A unit test of `remove` over `MemoryFiles`: a candidate whose path, or whose parent, is a symbolic link where the scan
  saw a directory is `Skipped` "changed since the scan", and the link's target is untouched.
- Component tests: `--json` alone exits 1 and removes nothing; `--json --dry-run` prints `would_remove` and `skipped`
  actions; `--json --force` prints the reaggregation events, then the document with `excludes_updated`.
- A component test: `--dry-run` runs no `git config` write and removes nothing.
- A component test: a broken link under `.claude/skills` is removed; an agent copy named by a stale block is removed.
- `cargo dev gate slice` exits 0.
