# Parallel workspace decisions

## Target selection and unknown names (decision:parallel-workspaces/target-selection)

**Decider:** product owner and engineering

**Open.** Choose defaults, literal-versus-pattern matching, no-match results, and unknown-target refusals across env and
worktree operations, without interpreting a typo as implicit all-target permission; this blocks finalizing
`product:parallel-workspaces/environment-lifecycle/environment-targets`,
`product:parallel-workspaces/coordinated-git/select-git-targets`,
`product:parallel-workspaces/safe-history/destructive-target-review`, and
`product:parallel-workspaces/stacked-history/declare-stack-order` under `decision:command-vocabulary`.

## Approval and changed plans (decision:parallel-workspaces/destructive-approval)

**Decider:** product owner and engineering

**Open.** Choose confirmation thresholds, unattended approval, and what happens when targets or loss sets change after
preview, keeping scope approval distinct from discard approval; this blocks finalizing
`product:parallel-workspaces/environment-lifecycle/destroy-environments`,
`product:parallel-workspaces/safe-history/destructive-target-review`, and
`product:parallel-workspaces/safe-history/remove-untracked-content`.

## Preflight and partial execution (decision:parallel-workspaces/partial-execution)

**Decider:** product owner and engineering

**Open.** Choose whole-selection versus per-target preflight, stopping versus continuing, and recovery obligations for
concurrent changes, interruption, and mid-run failure; this blocks finalizing
`product:parallel-workspaces/environment-lifecycle/environment-recovery`,
`product:parallel-workspaces/coordinated-git/integrate-selected-repositories`,
`product:parallel-workspaces/safe-history/history-operation-recovery`, and
`product:parallel-workspaces/stacked-history/resume-stack-rewrite`, not truthful reporting of completed effects.
