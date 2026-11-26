# Safe history engineering requirements

For `epic:parallel-workspaces/safe-history`.

## What already holds

- `ws checkout`, `ws reset`, and `ws clean` write the drift warning of
  [drift-warning.md](../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md), which this epic
  wires with each command; `epic:parallel-workspaces/repository-catalog` builds `drift::detect` and
  `drift::notice_line`.

## Abandonment evidence follows every moved ref (engineering:moved-ref-preservation)

Unpushed commits are measured against the configured upstream when its ref exists locally, otherwise against
`origin/<main>`; checkout also measures the env branch whenever HEAD is not on it. Dirt is staged or unstaged tracked
change and is checked before abandonment. A failed probe is that repository's error, never zero unpushed commits. Serves
`product:adopt-feature-history` and `product:reset-worktree-pointers`.

## Retain irreversible deletion evidence (engineering:clean-partial-accounting)

Clean preview and execution account for Git's actual removal units, translated output, special filenames, empty
directories, nested repositories, and nonzero exits after partial deletion. Parsing failure retains any known removed
paths and an uncertainty diagnosis, not an empty-success result. Serves `product:remove-untracked-content`.

## Isolated destructive verification (engineering:destructive-fixture-confinement)

Executable verification uses disposable repositories and workspace roots, never a selected source override against live
work; before/after assertions cover attached branches, retained commits, tracking, and untracked content independently.
Serves `product:history-operation-recovery`.
