# Integration

The one integration of a checkout onto a target ref that `ws pull` ([refresh.md](./refresh.md)) and `ws merge`
([merge.md](./merge.md)) run, and that `IntegrateCheckout` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), Domain,
names for a branch pin's advance. It never fetches and never leaves a merge or rebase in progress. Git calls go through
the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md), whose null
stdin keeps `git merge` and `git rebase` from opening an editor. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`; "WRR" is
`modules/workspace/internal/write_repo_repository.py`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/integration.rs — product:integrate-selected-repositories,
// decision:parallel-workspaces/coordinated-git/integration-conflicts, engineering:integration-recovery-evidence
pub(crate) enum Fallback { None, Merge, Rebase, NoFf }   // ff-only; --merge; pull --rebase; merge --no-ff

/// How one integration ended. The first five are the reference's; the rest are this epic's split of what the
/// reference reports as Diverged, each a registered deviation under integration-conflicts.
pub(crate) enum Integrated {
    UpToDate,                              // "up_to_date"
    FastForwarded { commits: u64 },        // "fast_forwarded"
    Merged { commits: u64 },               // "merged"
    Rebased { commits: u64 },              // "rebased"
    Diverged { ahead: u64, behind: u64 },  // "diverged": no fast-forward, and no fallback asked or the fallback
                                           // conflicted and was aborted cleanly
    MissingRef,                            // "missing_ref": the target names no commit
    DirtyBlocked,                          // "dirty_blocked": git refused before starting, for local changes
    AutostashFailed,                       // "autostash_failed": --autostash given and git refused or the stash
                                           // could not be restored
    AbortFailed { error: Box<RepoError> }, // "abort_failed": a conflicted merge or rebase whose --abort failed
}
pub(crate) fn integrate(programs: &dyn RunPrograms, tracer: &dyn CommandTracer, diagnostics: &mut Diagnostics<'_>,
    repo: &str, path: &Path, target: &str, fallback: Fallback, autostash: bool) -> Result<Integrated, GitFailure>;

// The IntegrateCheckout of standalone-pins.md is this function with Fallback from its PullMode
// (FfOnly -> None, Merge -> Merge, Rebase -> Rebase); its Integration enum maps Integrated one to one, and gains
// the four variants above, which advance passes through as Integrated like Diverged.

// termination.rs — GitOperation gains Integrate ("integration"), Abort ("abort of a merge or rebase")
```

## Interfaces

```text
# reference: WRR:820-878 (pull), 879-966 (merge), 968-1052 (diverged, counts, abort)
autostash        when true, "--autostash" goes right after the subcommand of every merge and rebase call below
commits          git rev-list --count HEAD..<target>, before anything moves; a failure is a warning naming the
                 repository and the range, and counts 0
ref check        git rev-parse --verify --quiet <target>^{commit}; non-zero -> MissingRef, nothing else runs
                 # ws merge runs this check first and reports skipped-missing-ref instead (merge.md)
fast-forward     Fallback::NoFf skips this step
                   h0 := git rev-parse HEAD
                   git merge [--autostash] --ff-only <target>
                   exit 0: git rev-parse HEAD == h0 -> UpToDate, else FastForwarded { commits }
                   non-zero: refused, classified below with "fallback allowed"
fallback         Fallback::None: no fallback; Merge: git merge [--autostash] <target>; Rebase: git rebase
                 [--autostash] <target>; NoFf: when commits is 0 -> UpToDate and nothing runs, else git merge
                 [--autostash] --no-ff <target>
                   exit 0 -> Merged { commits } or Rebased { commits }
                   non-zero: classified below
classify a refused call, in order:
  in progress    git rev-parse --verify --quiet MERGE_HEAD exits 0, or rev-parse --git-path rebase-merge or
                 rebase-apply names an existing directory: the call conflicted
                   git merge --abort, or git rebase --abort for a rebase
                     exit 0      -> Diverged { ahead, behind }
                     non-zero    -> AbortFailed { error: the abort's RepoError }; the checkout is left as git left
                                    it, and the error says so
  not started    the call refused before changing anything
                   the fast-forward step, and HEAD is an ancestor of the target
                   (git merge-base --is-ancestor HEAD <target> exits 0), or any fallback call:
                     autostash   -> AutostashFailed
                     dirty       -> DirtyBlocked, dirty as git::dirty answers it (tracked changes; untracked files
                                    git refused to overwrite count through git's own refusal)
                   otherwise, after the fast-forward step: the fallback runs when one is asked, else
                                    Diverged { ahead, behind }
  a fallback call that is refused and not started, with neither autostash nor a dirty tree, is Diverged
ahead, behind    git rev-list --count <target>..HEAD and git rev-list --count HEAD..<target>, after any abort; a
                 failure is a warning naming the repository and the target, and leaves both 0
a git that cannot be started is GitFailure::Internal; no other call above returns an error: each non-zero exit is
an outcome
```

```text
# what the reference does in each new case, for the parity record
MissingRef        pull: ff-only fails, Diverged 0/0 with a warning (WRR:842-851,1016-1039)
DirtyBlocked      Diverged with the real counts
AutostashFailed   Diverged with the real counts
AbortFailed       a warning "abort cleanup failed", then Diverged (WRR:1041-1052)
```

The span of each call is `git merge`, `git rebase`, `git rev-list`, or `git rev-parse`, with `winter.repo` as
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md), tracing,
gives it.
