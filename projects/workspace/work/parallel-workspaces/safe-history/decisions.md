# Safe history decisions

## Branch adoption scope and fallback (decision:parallel-workspaces/safe-history/adoption-policy)

**Decider:** product owner and engineering

`ws checkout` adopts a branch for a whole literal environment from local refs only, as the reference tool does: a
repository without the feature branch moves to its own main but still tracks the feature branch, `--new` only lifts the
refusal of a branch no repository knows, and a `{main}` target connects each env branch to main. Checking out a branch
means connecting to it, so adoption and tracking stay one step, while `ws reset` remains the way to move pointers alone.

## Pointer recovery and discard approval (decision:parallel-workspaces/safe-history/pointer-recovery)

**Decider:** product owner and engineering

Workspace keeps no recovery refs of its own, Git's reflog is the record, and one `--force` approves discarding both
tracked dirt and unpushed commits, as in the reference tool; `reset --hard` may overwrite untracked files in its way
because hard was asked for. Two registered deviations make the loss visible: each moved pointer's result names its
previous commit, and checkout without `--force` refuses the whole environment before any repository moves when an
untracked file blocks the target, instead of overwriting it.
