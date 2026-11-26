# Parallel workspace decisions

## Target selection and unknown names (decision:parallel-workspaces/target-selection)

**Decider:** product owner and engineering

Targets follow `product:target-patterns`, which keeps the reference tool's grammar, defaults, and no-match results.
Refusing an unknown literal environment before any effect, and reporting a missing worktree instead of skipping it
silently or failing part-way, in the structured result too except in `ws status`, whose published v1 schema admits no
member for it, are registered deviations: the reference crashes in some commands after earlier writes, and names a
skipped target in a warning at most.

## Approval and changed plans (decision:parallel-workspaces/destructive-approval)

**Decider:** product owner and engineering

One `--force` per command approves both its prompt and the discards its help names, as in the reference tool; prompt
thresholds are the component runtime's, and clean lists files again at removal. There is no separate scope approval,
because the reference's single override is what engineers and agents already use.

## Preflight and partial execution (decision:parallel-workspaces/partial-execution)

**Decider:** product owner and engineering

Each command keeps the reference tool's policy: checkout, reset, merge tokens, and restack refuse the whole selection
before running serially; clean stops at its first failure; fetch, pull, push, init, and destroy attempt every target and
fail if any did. Nothing rolls back. As a registered deviation, a multi-target command that stops or fails part-way
names its completed, failed, and unattempted targets, which the reference's checkout and reset do not.
