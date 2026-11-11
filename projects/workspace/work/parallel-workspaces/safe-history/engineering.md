# Safe history engineering requirements

For `epic:parallel-workspaces/safe-history`.

## Abandonment evidence follows every moved ref (engineering:moved-ref-preservation)

Safety evidence covers both current HEAD and any separate env branch the operation will overwrite, with configured
upstream distinguished from a locally present upstream ref. Failed history probes cannot count as zero unpublished
commits. Serves `product:adopt-feature-history` and `product:reset-worktree-pointers`.

## Validate against concurrent Git changes (engineering:destructive-effect-revalidation)

The approved target identities, safety-relevant ref positions, and filesystem confinement remain checked at the mutation
boundary across processes; changed evidence cannot authorize a different destructive effect under old approval. Serves
`product:destructive-target-review` and `product:history-operation-recovery`.

## Retain irreversible deletion evidence (engineering:clean-partial-accounting)

Clean preview and execution account for Git's actual removal units, translated output, special filenames, empty
directories, nested repositories, and nonzero exits after partial deletion. Parsing failure retains any known removed
paths and an uncertainty diagnosis, not an empty-success result. Serves `product:remove-untracked-content`.

## Isolated destructive verification (engineering:destructive-fixture-confinement)

Executable verification uses disposable repositories and workspace roots, never a selected source override against live
work; before/after assertions cover attached branches, retained commits, tracking, and untracked content independently.
Serves `product:history-operation-recovery`.
