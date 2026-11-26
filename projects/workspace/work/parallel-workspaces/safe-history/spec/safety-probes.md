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
// dirty: a staged or unstaged tracked change; untracked files never count. unpushed: the commits reachable from
// the moving ref and not from its safety ref. dirty is git::dirty as built, called with wt.repo and wt.path: it has
// no wrapper here, since git::dirty already holds the name at the crate root
pub fn has_ref(ports: &Ports<'_>, wt: &Selected, rev: &str) -> Result<bool, GitFailure>;
pub fn safety_ref(ports: &Ports<'_>, wt: &Selected, branch: Branch) -> Result<String, GitFailure>;
pub fn unpushed(ports: &Ports<'_>, wt: &Selected, from: &str, safety: &str) -> Result<u64, GitFailure>;
pub fn active_branch(ports: &Ports<'_>, wt: &Selected) -> Result<Option<String>, GitFailure>;
pub fn commit(ports: &Ports<'_>, wt: &Selected, rev: &str) -> Result<Option<Sha>, GitFailure>;
pub enum Branch<'b> { Head, Named(&'b str) }
// termination.rs: GitOperation gains AbandonmentProbe ("abandonment probe"), UntrackedProbe ("untracked probe"),
// Reset ("reset"), Clean ("clean"); the spans "git rev-list", "git ls-files", "git ls-tree", "git reset", and
// "git clean" join git.rs's set, each call's span "git <subcommand>". Rebase and the "git rebase" and
// "git merge-base" spans are not this epic's: they are ws restack's (stacked-history) and integration's
// (coordinated-git)
```

Visibility: `has_ref`, `safety_ref`, `unpushed`, `active_branch`, `commit`, and `Branch` are `pub` inside their private
module and re-exported by name from `lib.rs`, each module's names on a `pub use` line of their own, with every type
their signatures name (`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that
lands each tests it before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without
an `allow` (`context/standards/rust.md`). `Selected` is `pub` and re-exported by
[target-patterns.md](../../coordinated-git/spec/target-patterns.md), Domain; `Ports`, `GitFailure`, and `Sha` already
are. Everything else here stays `pub(crate)`.

The operation each git call of this epic names when it fails as a repo error:

| Call                                                                                                | A repo error names                                                                     |
| --------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| `dirty`                                                                                             | `TrackedStatus`, as built                                                              |
| `upstream` and `active_branch`: `git symbolic-ref`, `git config --get`                              | `TrackingRead`, of `epic:parallel-workspaces/coordinated-git`                          |
| `unpushed`                                                                                          | `AbandonmentProbe`                                                                     |
| checkout's untracked check: `git ls-files`, `git ls-tree` ([checkout.md](./checkout.md), Preflight) | `UntrackedProbe`                                                                       |
| checkout's `git checkout` ([checkout.md](./checkout.md), Execution, step 2)                         | `Checkout`, as built                                                                   |
| checkout's tracking writes (Execution, step 3)                                                      | `SetUpstream`, of `epic:parallel-workspaces/environment-lifecycle`                     |
| checkout's `push.default` write (Execution, step 4)                                                 | `Config`, as [push-default.md](../../environment-lifecycle/spec/push-default.md) gives |
| `git reset` ([reset.md](./reset.md), Execution)                                                     | `Reset`                                                                                |
| `git clean` ([clean.md](./clean.md), Git's clean)                                                   | `Clean`                                                                                |

## Interfaces

```text
# reference: modules/workspace/worktree_safety.py:17-50, internal/write_repo_repository.py:256-323
dirty(wt)            git::dirty(programs, tracer, wt.repo, wt.path), as built: git status --porcelain
                     --untracked-files=no, GIT_OPTIONAL_LOCKS=0; output non-empty
                     -> true; a non-zero exit is the repo error of the call, never "clean"
has_ref(wt, rev)     git rev-parse --verify --quiet <rev>; exit 0 -> true, any other exit -> false
                     # a tag, a remote-tracking ref, or a commit id all count: the reference does not restrict the kind
upstream(wt, b)      b = Head: the branch `git symbolic-ref --quiet HEAD` names, stripped of refs/heads/; exit 1
                     (detached) -> None; an unborn HEAD names its branch, which has no upstream configured; another
                     non-zero exit is the repo error of the call
                     remote := git config --get branch.<branch>.remote; merge := git config --get branch.<branch>.merge
                     either unset (exit 1) -> None; another non-zero exit is the repo error of the call
                     else name := "<remote>/<merge without a leading refs/heads/>"
                     git rev-parse --verify --quiet <name>^{commit} exit 0 -> Some(name), else None
safety_ref(wt, b)    upstream(wt, b), else "origin/<main_branch>" (the token expansion of origin/{main})
                     # measured against what the pointer moves away from, never the destination
unpushed(wt, f, s)   git rev-list --count <f> ^<s>; stdout as a decimal count
                     a non-zero exit, e.g. a missing origin/<main_branch>, is the repo error of the call: the
                     command ends with it before any effect, never reading zero (product:adopt-feature-history)
active_branch(wt)    as upstream's Head branch read: Some(name), also for an unborn HEAD, or None when HEAD is detached
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
