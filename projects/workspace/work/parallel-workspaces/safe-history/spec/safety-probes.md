# Safety probes

The read-only probes `ws checkout` and `ws reset` run before moving a pointer, and the record of where a pointer stood
before it moved. [checkout.md](./checkout.md) and [reset.md](./reset.md) call them; `ws restack` has its own guards
([restack-plan.md](../../stacked-history/spec/restack-plan.md)). Every git call goes through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md) with `cwd`
the worktree's path; a git that cannot be started is `InternalError`. `RepoError` and `Termination` are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md).
Ref tokens expand as [ref-tokens.md](../../coordinated-git/spec/ref-tokens.md) gives. Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`.

## Domain

```rust
// crates/core/src/history.rs — engineering:moved-ref-preservation
// A selected worktree is the value target-patterns.md yields (../../coordinated-git/spec/target-patterns.md): env,
// repository name, worktree path, and the repository's configured main branch. Pinned projects never reach here.
pub(crate) enum Risk {
    Dirty,                    // staged or unstaged tracked change; untracked files never count
    Abandonment,              // commits reachable from the moving ref and not from its safety ref
}
pub(crate) fn dirty(ports: &Ports<'_>, wt: &Selected) -> Result<bool, GitFailure>;          // git::dirty, as built
pub(crate) fn has_ref(ports: &Ports<'_>, wt: &Selected, rev: &str) -> Result<bool, GitFailure>;
pub(crate) fn safety_ref(ports: &Ports<'_>, wt: &Selected, branch: Branch) -> Result<String, GitFailure>;
pub(crate) fn unpushed(ports: &Ports<'_>, wt: &Selected, from: &str, safety: &str) -> Result<u64, GitFailure>;
pub(crate) fn active_branch(ports: &Ports<'_>, wt: &Selected) -> Result<Option<String>, GitFailure>;
pub(crate) fn commit(ports: &Ports<'_>, wt: &Selected, rev: &str) -> Result<Option<Sha>, GitFailure>;
pub(crate) enum Branch<'b> { Head, Named(&'b str) }
// termination.rs: GitOperation gains AbandonmentProbe ("abandonment probe"), Reset ("reset"), Clean ("clean"),
// Rebase ("rebase"); the spans "git rev-list", "git reset", "git clean", "git rebase", "git merge-base" join git.rs's set
```

## Interfaces

```text
# reference: modules/workspace/worktree_safety.py:17-50, internal/write_repo_repository.py:256-323
dirty(wt)            git status --porcelain --untracked-files=no, GIT_OPTIONAL_LOCKS=0 (git::dirty); output non-empty
                     -> true; a non-zero exit is the repo error of the call, never "clean"
has_ref(wt, rev)     git rev-parse --verify --quiet <rev>; exit 0 -> true, any other exit -> false
                     # a tag, a remote-tracking ref, or a commit id all count: the reference does not restrict the kind
upstream(wt, b)      b = Head: the branch `git symbolic-ref --quiet HEAD` names, stripped of refs/heads/; exit 1
                     (detached or unborn) -> None
                     remote := git config --get branch.<branch>.remote; merge := git config --get branch.<branch>.merge
                     either unset -> None; else name := "<remote>/<merge without a leading refs/heads/>"
                     git rev-parse --verify --quiet <name>^{commit} exit 0 -> Some(name), else None
safety_ref(wt, b)    upstream(wt, b), else "origin/<main_branch>" (the token expansion of origin/{main})
                     # measured against what the pointer moves away from, never the destination
unpushed(wt, f, s)   git rev-list --count <f> ^<s>; stdout as a decimal count
                     a non-zero exit, e.g. a missing origin/<main_branch>, is the repo error of the call: the
                     command ends with it before any effect, never reading zero (product:adopt-feature-history)
active_branch(wt)    as upstream's Head branch read: Some(name), or None when HEAD is detached or unborn
commit(wt, rev)      git rev-parse --verify --quiet <rev>^{commit}; exit 0 -> Some(sha), else None
probe order          per worktree: dirty first; a dirty worktree is refused with Dirty and not measured further
```

### Previous position

```text
# decision:parallel-workspaces/safe-history/pointer-recovery — registered deviation: the reference reports no
# previous position; Winter keeps no recovery ref of its own and Git's reflog stays the record
captured             in each worktree immediately before its first mutating call, never at preflight
  head               commit(wt, "HEAD"): the commit HEAD named, None for an unborn HEAD
  branch             ws checkout only: commit(wt, "refs/heads/<env>"), None when that branch did not exist
reported             with each moved worktree's outcome in the text result and in the JSON (checkout.md, reset.md);
                     also on a worktree whose later step failed after its pointer had moved
```
