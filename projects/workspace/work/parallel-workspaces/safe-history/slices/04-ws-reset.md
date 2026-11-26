---
name: ws-reset
depends-on:
  - safety-probes
  - epic:parallel-workspaces/coordinated-git
---

# The `ws reset` command

Adds `ResetMode`, `ResetResult`, `ResetEvent`, `RepoReset`, and the `ws reset PATTERNS... REF` command of
[reset.md](../spec/reset.md): selection through `targets::select` with `EnvRule::Named`, REF expanded per repository by
`ref_tokens::expand` of `epic:parallel-workspaces/coordinated-git`, the `--hard` prompt, an all-or-nothing preflight
over the probes of `safety-probes`, then `git reset` one worktree at a time with each moved worktree's previous commit
reported. Tracking and attachment are never touched. Its rows are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its
prompt the `ws reset --hard` row of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Confirmation prompts. Delivers `product:reset-worktree-pointers`, the reset bullets of
`product:destructive-target-review` and `product:history-operation-recovery`, and
`engineering:destructive-fixture-confinement` for reset, under `decision:parallel-workspaces/destructive-approval`,
`decision:parallel-workspaces/partial-execution`, and `decision:parallel-workspaces/safe-history/pointer-recovery`,
whose previous commit, `failed`, and `unattempted` are registered deviations.

## Change

- `crates/core/src/history/reset.rs` holds the types and the run: the drift warning, selection, nothing matched, the
  prompt before every refusal check, the ambiguous-sha, expansion, and missing-ref refusals, and under `--hard` without
  `--force` the dirty and abandonment refusals, all before any write and also under `--dry-run`.
- Execution captures `commit(wt, "HEAD")` immediately before each `git reset --soft|--mixed|--hard`; the first failure
  is `failed` with its error, and every later target `unattempted` and untouched; nothing rolls back.
- `crates/core/src/commands/ws/reset.rs` registers the command with `--soft`, `--mixed`, `--hard`, `--force`,
  `--dry-run`, and `--json`, writes the `target_missing` events, then the shape C events or the text table and its
  refused, complete, or stopped lines, and exits as Running gives.
- `crates/core/topics/ws/reset.md`, referenced from the `ws` group topic: reset acts on the worktrees its patterns
  select, never a pinned one, keeps tracking, and is not `ws checkout`.

## Acceptance

- Component tests through `InProcess` over a workspace root and fixture remote the test creates: `--soft` keeps the
  index and working files, `--mixed` resets the index and keeps the files, and `--hard` resets all three; each leaves
  `branch.<env>.*` unchanged, a detached HEAD detached, and an untracked file in place, and names the old HEAD as
  `previous`.
- A component test: `--hard` overwrites an untracked file in the way of a tracked file at REF and leaves another
  untracked file in place.
- Component tests: `--hard` refuses a worktree with tracked dirt as `refused-dirty` and one with commits its upstream
  lacks as `refused-abandonment`, even when REF holds those commits, and moves neither worktree of the run; `--force`
  resets both; `--mixed` resets a dirty worktree without refusing.
- A component test: a REF missing in one of two selected worktrees is `refused-missing-ref` there, nothing moves in
  either, and the command exits 1, also under `--force`.
- Component tests: `ws reset 'alpha/*' abc1234` over two worktrees is `refused-ambiguous-sha` for both, also under
  `--force`; over one worktree it resets; `refs/heads/abc1234` resets both; `origin/{mian}` exits 1 with `RefToken`
  before any git call.
- Component tests: `--hard` over two worktrees without `--force` or `--dry-run` writes the preview and "Continue?" on
  stdout, before the first event under `--json`; answering no, or ending input, exits 1 with nothing moved; over one
  worktree it never prompts.
- A component test: `--dry-run` reports each target `reset` with its expanded ref, and every ref and file is unchanged.
- A component test: with `index.lock` held in the second of three worktrees, the first is `reset` with `previous`, the
  second `failed` with Git's cause, the third `unattempted` and unchanged, and the command exits 1.
- A component test: a pinned worktree the pattern matches does not move; an absent worktree is a `target_missing` event
  before `reset_started`.
- `cargo dev gate slice` exits 0.
