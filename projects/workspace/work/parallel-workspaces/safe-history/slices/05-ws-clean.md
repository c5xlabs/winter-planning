---
name: ws-clean
depends-on:
  - git-clean
  - epic:parallel-workspaces/coordinated-git
---

# The `ws clean` command

Adds `RepoClean`, `CleanFailure`, `CleanReport`, `CleanEvent`, `preview`, `clean`, `run`, and the `ws clean PATTERNS...`
command of [clean.md](../spec/clean.md), Running and Result: selection through `targets::select` with `EnvRule::Named`,
the preview from `list` of `git-clean`, the confirmation, then `remove` one worktree at a time, each worktree's outcome
written as it is reached, the report naming what Git actually removed and stopping at the first failed worktree. Its
rows are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its
prompt the `ws clean` row of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Confirmation prompts. Delivers `product:remove-untracked-content`'s approval, result, and failure bullets, the clean
bullets of `product:destructive-target-review` and `product:history-operation-recovery`, and
`engineering:destructive-fixture-confinement` for clean, under `decision:parallel-workspaces/destructive-approval` and
`decision:parallel-workspaces/partial-execution`, whose `unattempted` list and kept partial outcome are registered
deviations.

## Change

- `crates/core/src/history/clean.rs` gains the report types, `CleanEvent`, `preview`, `clean`, and `run`, which takes
  the start path: root discovery and the load, then the drift warning of clean.md, Running, step 2, through
  `drift::detect` and `drift::notice_line` before selection; a load, selection, or preview failure is `CleanError` and
  ends the command before anything is removed; the prompt goes through its `confirm` closure; `remove` visits every
  target, also one the preview showed empty, and a failed call or strict parse stops the run with the paths recognised
  and the targets not reached; each target's `RepoClean` goes to the sink as its call returns.
- `crates/core/src/commands/ws/clean.rs` registers the command with `--force`, `--dry-run`, and `--json`; `--json`
  without `--force` or `--dry-run` is `HistoryError::JsonNeedsForce` before the call; it builds the `confirm` closure
  over `confirm.rs`, which renders the preview and confirms, even for one worktree, and the event sink, makes its one
  call to `run`, maps a `CleanError` to its `Termination` as clean.md, Domain, gives, and writes the `target_missing`
  events from the started event's `missing`, then each shape C event or its text as the sink receives it.
- `crates/core/topics/ws/clean.md`, referenced from the `ws` group topic: clean removes untracked files from the
  worktrees its patterns select, never ignored files or a pinned worktree, cannot be undone, and is not the top-level
  `clean` of `epic:running-environments/resource-provisioning`.

## Acceptance

- Component tests through `InProcess` over a workspace root the test creates: `--dry-run` lists each worktree's
  untracked paths and their totals and removes nothing; `--force` removes them and reports the same paths, while an
  ignored file and a tracked modification stay.
- Component tests: without `--force` or `--dry-run`, the preview names every path per worktree and that this cannot be
  undone, then "Continue?", also for one worktree; answering no, or ending input, exits 1 with nothing removed.
- A component test: `--json` without `--force` or `--dry-run` exits 1 with `JsonNeedsForce` and no git call, as a
  recording program port shows.
- Component tests: a selection with nothing untracked writes "Nothing to clean" without a prompt and exits 0; a pattern
  matching nothing writes its one line and exits 0; under `--json` each writes the events Result gives.
- A component test: `--json --force` writes `clean_started`, one `repo_clean` per target, an empty one included, and
  `clean_completed` with the total.
- A component test: over two worktrees, with a recording program port, the first worktree's `repo_clean` is on stdout
  before the second worktree's `git clean -fd` starts.
- A component test: a `confirm` closure that answers yes after creating an untracked file in the first worktree; that
  file is removed and named in its `repo_clean` paths, though the preview did not list it.
- A component test: with the second of three worktrees failing after removing some paths, its `repo_clean` holds the
  paths removed, `failure` names it and the third as unattempted, the third keeps its untracked files, and the command
  exits 1; the text says the listed paths are gone for good.
- A component test: a pinned worktree the pattern matches keeps its untracked files.
- A component test: over a workspace with a drifted project, the drift notice of
  [drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md) is written on
  stderr before the result, also under `--json`, through `drift::detect` and `drift::notice_line`.
- `cargo dev gate slice` exits 0.
