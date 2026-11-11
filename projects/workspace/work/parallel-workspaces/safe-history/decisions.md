# Safe history decisions

## Branch adoption scope and fallback (decision:parallel-workspaces/safe-history/adoption-policy)

**Decider:** product owner and engineering

**Open.** Choose env-wide versus narrower adoption, missing-feature fallback, and explicit new-branch intent without
combining ordinary reset with tracking changes; this blocks finalizing `product:adopt-feature-history` and its scope in
`product:destructive-target-review`.

## Pointer recovery and discard approval (decision:parallel-workspaces/safe-history/pointer-recovery)

**Decider:** product owner and engineering

**Open.** Choose retained recovery references or equivalent recovery information and how explicit discard consent
applies to unpublished commits, obstructing untracked files, and tracked dirt; this blocks finalizing
`product:reset-worktree-pointers` and `product:history-operation-recovery`, not protection by default.
