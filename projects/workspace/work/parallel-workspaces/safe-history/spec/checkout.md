# Checkout

The `ws checkout` command: adopting a feature branch into every non-pinned worktree of one environment. Its options are
the `ws checkout` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands; its
JSON document, which this leaf owns, is shape A of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md). The probes and
the previous position are those of [safety-probes.md](./safety-probes.md); the environment and its worktrees are
selected as [target-patterns.md](../../coordinated-git/spec/target-patterns.md) gives for a literal environment; tokens
expand as [ref-tokens.md](../../coordinated-git/spec/ref-tokens.md); `push.default` is written as
[push-default.md](../../coordinated-git/spec/push-default.md) gives; the command is serial, as
[fan-out.md](../../coordinated-git/spec/fan-out.md) lists. The drift warning is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the
binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/history/checkout.rs — product:adopt-feature-history,
// decision:parallel-workspaces/safe-history/adoption-policy
pub(crate) enum CheckoutResult {
    ResetFeature,             // "reset-feature": moved to origin/<feature>
    ResetMain,                // "reset-main": moved to origin/<main_branch>, tracking origin/<feature>
    RefusedDirty,             // "refused-dirty"            bypassed by --force
    RefusedAbandonment,       // "refused-abandonment"      bypassed by --force
    RefusedUnknownBranch,     // "refused-unknown-branch"   never bypassed; lifted by --new
    RefusedMissingRef,        // "refused-missing-ref"      never bypassed
    RefusedUntracked,         // "refused-untracked"        bypassed by --force; registered deviation (Preflight)
    Failed,                   // "failed"                   registered deviation (Execution)
    Unattempted,              // "unattempted"              registered deviation (Execution)
}
pub(crate) struct RepoCheckout {
    pub repo_name: String,
    pub result: CheckoutResult,
    pub previous: Option<Previous>,   // Some once the worktree's pointer moved (safety-probes.md)
    pub error: Option<String>,        // Failed: the repo error's message and git's stderr; RefusedUntracked: the
                                      // blocking paths
}
pub(crate) struct Previous { pub head: Option<Sha>, pub branch: Option<Sha> }
pub(crate) struct CheckoutReport { pub env: String, pub feature_branch: String, pub aborted: bool, pub repos: Vec<RepoCheckout> }
pub(crate) fn checkout(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, workspace: &Workspace,
                       env: &str, feature_branch: &str, force: bool, new: bool) -> Result<CheckoutReport, Termination>;
```

## Interfaces

### Running `ws checkout ENV FEATURE_BRANCH`

```text
# reference: modules/workspace/handlers/workspace_handler.py:411-471, env_checkout_service.py:135-277
order
  1. root discovery and the load; a failure is its ConfigError, exit 1
  2. the drift warning
  3. ENV passes env_name::check (../../environment-lifecycle/spec/env-names.md) or the command is refused;
     then ENV as a literal environment (target-patterns.md): one neither registered nor present is refused before any git
     call; its worktrees are every non-pinned project's <root>/<ENV>/<repo>, in configuration order; a missing one is
     warned about and skipped (target-patterns.md). targets := the rest
  4. per target, before any git call: feature_ref := expand("origin/<FEATURE_BRANCH>"), main_ref := "origin/<main_branch>";
     an unknown token refuses the whole command (ref-tokens.md)
  5. preflight (below): any refusal -> the report with aborted true and only the refused repos; nothing is written
  6. execution (below), serially in target order
  7. the result: text or the document; exit 0 when no repo is refused, failed, or unattempted; else 1 (Reported)
no network: no fetch runs; every ref is read from the local store
no confirmation, at any count (results-and-exit-codes.md, Confirmation prompts)
```

### Preflight

```text
# all-or-nothing over targets (decision:parallel-workspaces/partial-execution); refusals in target order
have(wt)              := has_ref(wt, feature_ref)
unknown branch        targets non-empty, not --new, and have(wt) false for every target
                      -> every target refused-unknown-branch; no other check runs
per target, in order:
  missing ref         not have(wt) and not has_ref(wt, main_ref) -> refused-missing-ref; not checked further
  unless --force:
    dirty             dirty(wt) -> refused-dirty
    HEAD              unpushed(wt, "HEAD", safety_ref(wt, Head)) > 0 -> refused-abandonment
    env branch        active_branch(wt) is not ENV and has_ref(wt, "refs/heads/<ENV>"):
                      unpushed(wt, "refs/heads/<ENV>", safety_ref(wt, Named(ENV))) > 0 -> refused-abandonment
    untracked         untracked := git ls-files --others --exclude-standard -z; incoming := git ls-tree -r
                      --name-only -z <target_rev> (target_rev as Execution gives it); a path of untracked that
                      equals a path of incoming, that is a leading directory of one (u + "/" starts it), or that
                      lies under one (i + "/" starts u) -> refused-untracked, error naming those paths
                      # ignored files are not listed, so Git's expendable ignored files never refuse
                      # decision:parallel-workspaces/safe-history/pointer-recovery — registered deviation: the
                      # reference always forces Git and overwrites the file
a target takes its first refusal only; the checks after it are not run for that target
a probe's repo error ends the command with that RepoError, exit 1, before any effect
```

### Execution

```text
# per target, in order; the env branch moves before tracking is written, so a failure between the two leaves the
# worktree on its old tracking
target_rev            have(wt) ? feature_ref : main_ref
  1. previous := Previous { head: commit(wt, "HEAD"), branch: commit(wt, "refs/heads/<ENV>") }
  2. --force:    git checkout --force -B <ENV> --no-track <target_rev>
     otherwise:  git checkout -B <ENV> --no-track <target_rev>
     # without --force, preflight already refused tracked dirt and untracked files in the way; Git's own refusal
     # remains a backstop for a file created since
  3. git config branch.<ENV>.remote origin
     git config branch.<ENV>.merge refs/heads/<feature_ref after its first "/">
     # written directly, never through --set-upstream-to, so a branch origin does not have yet can be tracked
  4. push.default=upstream (push-default.md)
  -> reset-feature when have(wt), else reset-main; previous reported
step 2 fails      -> failed; previous not reported: nothing moved
step 3 or 4 fails -> failed, previous reported: the pointer moved
either stops the run: every later target is unattempted and untouched; nothing rolls back
# decision:parallel-workspaces/partial-execution — registered deviation: the reference raises the repo error with no
# report of the targets already moved or never reached
```

### Result

```text
--json  one document, written once at the end:
  {"env": <ENV>, "feature_branch": <FEATURE_BRANCH as typed>, "aborted": bool, "repos": [repo...]}
  repo: {"repo_name", "result", "previous", "error"}
    previous: {"head": sha|null, "branch": sha|null} when the pointer moved, else null    # registered deviation
    error:    message text on failed, the blocking paths on refused-untracked, else null  # registered deviation
  aborted true:  preflight refused; repos holds only the refused repos
  aborted false: repos holds every target, in order
text    a table of REPO and RESULT, one row per entry of repos, then the previous commits of every moved repo
  aborted:  that nothing changed, the blocking paths of each refused-untracked repo, and one hint: fetch or --new for
            unknown-branch alone; fetch when any missing-ref; --force otherwise
  complete: the env, the feature branch, the count moved to the feature and, when any, the count started from main
  stopped:  the repo it stopped on with its error, the repos already moved, and the unattempted ones
```
