# Restack plan

The plan of `ws restack`: reading the chain, refusing it, or freezing each link's boundary before anything moves, and
the `--dry-run` result. Its options and combination rules, the literal chain-name shape included, are the `ws restack`
rows of [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); its
JSON documents, which this leaf and [restack-execution.md](./restack-execution.md) own, are shape A of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md). BASE and
`--cut` expand per repository as [ref-tokens.md](../../coordinated-git/spec/ref-tokens.md) gives; the command is serial,
as [fan-out.md](../../coordinated-git/spec/fan-out.md) lists. Every git call goes through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md) with `cwd`
the worktree `<root>/<env>/<repo>`. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
`winter-workspace` at `127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/restack.rs — product:declare-stack-order, product:review-replay-boundaries,
// decision:parallel-workspaces/stacked-history/boundary-policy
pub struct Chain { pub envs: Vec<String>, pub base: String, pub cut: Option<String> }  // envs top of stack first
/// The ws restack rules of command-surface.md, Combination rules, in its order, before anything is read: tokens are
/// ENV... BASE as parsed (at least one, the parser's own rule); the first violation is the one returned.
pub fn chain(tokens: &[String], cut: Option<String>) -> Result<Chain, ChainError>;
pub enum Refusal {
    MissingRef,               // "refused-missing-ref"
    Dirty,                    // "refused-dirty"
    RebaseInProgress,         // "refused-rebase-in-progress"
    DetachedHead,             // "refused-detached-head"
    InvertedOrder,            // "refused-inverted-order"
    UnknownBoundary,          // "refused-unknown-boundary"
    CutNotAncestor,           // "refused-cut-not-ancestor"
}
pub struct PlanRefusal {
    pub result: Refusal, pub env: String, pub predecessor: String,
    pub repos: Vec<String>,           // sorted; empty for a repeat and for the env or BASE MissingRef
    pub element: Option<String>,      // MissingRef: the env, BASE as typed, or "--cut"; InvertedOrder: the repeated cut
}
pub enum Source { ForkPoint, Cut }              // "fork-point" | "cut"
pub struct LinkRepo { pub repo_name: String, pub boundary: Option<Sha>, pub source: Option<Source> }
pub struct Link { pub env: String, pub predecessor: String, pub repos: Vec<LinkRepo> }
pub enum Plan { Refused(Vec<PlanRefusal>), Links(Vec<Link>) }  // Links bottom link first
pub fn plan(ports: &Ports<'_>, workspace: &Workspace, chain: &Chain) -> Result<Plan, RestackError>;

// the module's own errors (changes.md, An error); lib.rs re-exports both with the types their variants carry
pub enum ChainError {                 // a command error, exit 1, nothing read; reference: modules/workspace/command.py
    OneToken,                         // "restack requires at least one ENV and a trailing BASE"          (:704)
    EmptyElement,                     // "Empty chain element is not allowed"                             (:116)
    GlobElement { element: String },  // "Invalid chain element '<element>' — restack takes literal env names, not
                                      //  globs"                                                           (:118)
    ScopedElement { element: String },// "Invalid chain element '<element>' — restack has no '<env>/<repo>' scoping"
                                      //                                                                   (:120)
    EmptyBase,                        // "Empty BASE is not allowed"                                       (:709)
}
// termination.rs — CommandError gains Chain(#[from] ChainError), rendered transparently
pub enum RestackError {               // plan's failure; the run ends before any write
    RefToken(RefTokenError),          // Refusals, step 1a (ref-tokens.md)
    Repo(Box<RepoError>),             // Reads: a repo error of dirty or branch
    Internal(Box<InternalError>),     // Reads: a git that cannot be started
}
// the command file maps: ChainError -> Command(Chain); RestackError::RefToken -> Command(RefToken),
// Repo -> Repo, Internal -> Internal, each exit 1, the box handed on as it is
// termination.rs — GitOperation gains Rebase ("rebase"), the operation of the rebase's repo error
// (restack-execution.md); branch's repo error is TrackingRead of ../../coordinated-git/spec/push.md
// spans: restack.rs names "git merge-base" and restack/execute.rs "git rebase", beside git.rs's "git rev-parse"
// and "git status"; each other call's span is "git <subcommand>"
```

Visibility: `chain`, `Chain`, `ChainError`, `plan`, `Plan`, `Refusal`, `PlanRefusal`, `Source`, `LinkRepo`, `Link`,
`RestackError`, and the reads `tip`, `ancestor`, `fork`, `rebasing`, and `branch` of Reads below are `pub` inside their
private module and re-exported by name from `lib.rs`, each module's names on a `pub use` line of their own, with every
type their signatures name (`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice
that lands each tests it before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused
without an `allow` (`context/standards/rust.md`). Everything else here stays `pub(crate)`.

## Interfaces

### Reads

```text
# reference: modules/workspace/env_restack_plan_service.py:60-200, internal/read_repo_repository.py:676-757
links         (envs[i], envs[i+1]) for each i, the last (envs[n-1], BASE): argument order, top first
repos         every non-pinned project, in configuration order; no environment lookup and no registry read:
              the worktree is <root>/<env>/<repo> whether or not it exists
pred_ref      the bottom link: BASE as Refusals, step 1a, expanded it for the repo; every other link:
              refs/heads/<predecessor>
env_ref       refs/heads/<env>                     # engineering:stack-boundary-integrity: a tag never stands in
tip(wt, ref)  git rev-parse --verify --quiet <ref> -> the sha, or None on any non-zero exit, a missing worktree, or
              a directory that is no repository
participates  env tip and predecessor tip both Some; otherwise the repo is skipped in that link
ancestor(a,b) git merge-base --is-ancestor <a> <b>: exit 0 -> true; any other exit, or a missing worktree -> false
fork(wt)      git merge-base --fork-point <pred_ref> <env_ref>: exit 0 with non-empty stdout -> the sha; else None
rebasing(wt)  git rev-parse --git-path rebase-merge, then rebase-apply: either path exists -> true
dirty(wt)     git::dirty, as built (safety-probes.md's dirty): tracked changes only, no optional lock taken
branch(wt)    git symbolic-ref --quiet HEAD stripped of refs/heads/; exit 1 -> detached; any other exit -> the
              repo error of the call, TrackingRead
spans         each call opens the span "git <subcommand>" with winter.repo, as repository-catalog's
              location-listing.md, tracing, gives
a git that cannot be started is RestackError::Internal; dirty and branch reading a repo error end the run with it as
              RestackError::Repo, exit 1
```

### Refusals

```text
# all-or-nothing: any refusal and no link is built; refusals are listed in the order below
1. repeated      the first value of envs ++ [BASE] that occurs twice: the first link whose env is that value,
                 InvertedOrder, repos [], element None; else --cut equal to an env or BASE: the bottom link,
                 InvertedOrder, element the cut. A repeat is the only refusal reported; nothing is read
1a. tokens       for every non-pinned repo, in configuration order: expand(BASE) and, with --cut, expand(cut)
                 (ref-tokens.md); a RefTokenError ends the run as RestackError::RefToken, its command error, exit 1,
                 before any git call
                 # product:parallel-workspaces/ref-tokens; registered deviation,
                 # decision:parallel-workspaces/stacked-history/token-expansion: the reference expands them during
                 # its reads, and the cut only in participating repos
2. reads         every link, every repo
3. missing refs  each link, top first, whose env tip is None in every repo -> MissingRef, repos [], element the env
                 the bottom link, when some repo has its env tip and every such repo lacks the predecessor tip
                 -> MissingRef, repos [], element BASE as typed
4. per link, top first, over its participating repos:
   safety        Dirty, then RebaseInProgress, then DetachedHead (HEAD not on refs/heads/<env>), each one refusal
                 naming its repos
   boundaries    the bottom link with --cut: per repo, cut_tip := tip(the cut expanded in 1a); None -> MissingRef
                 with element "--cut"; ancestor(cut_tip, env tip) (applies) or ancestor(predecessor tip, env tip)
                 (already done)
                 -> boundary cut_tip, Cut; neither -> CutNotAncestor; MissingRef before CutNotAncestor
                 every other link: fork(wt); None -> UnknownBoundary naming the repos; else boundary, ForkPoint
                 # no merge-base fallback; --cut is never a boundary for an upper link
   inversion     with any boundary, and no repo whose env tip equals its boundary: when in every repo with a
                 boundary the env tip differs from the predecessor tip and ancestor(env tip, predecessor tip)
                 -> InvertedOrder naming those repos
                 # declared limit: a chain typed backwards in any other shape is not detected
                 # (product:declare-stack-order)
```

### Plan

```text
links         reversed: the bottom link first, the order of execution
link repos    every non-pinned repo in configuration order; a skipped one carries boundary and source None
frozen        boundaries are read once, here; execution reads new bases afresh (restack-execution.md)
```

### Result

```text
# built by commands/ws/restack.rs from the domain's values, documents and text alike (changes.md, A command)
--json, one document, no discriminator field (output-formats.md)
  refused   {"refusals": [{"result", "env", "predecessor", "repos": [...], "element": str|null}...]}; exit 1
  --dry-run {"links": [{"env", "predecessor", "repos": [{"repo_name", "boundary": sha|null, "source":
            "fork-point"|"cut"|null}...]}...]}; exit 0, no write call at all
text
  refused   that the restack was refused with no change, then per refusal its result, "<env> onto <predecessor>",
            its repos, its element when there is one, and how to proceed (product:preserve-unrelated-work):
              Dirty               commit or stash the tracked changes
              RebaseInProgress    finish it with git rebase --continue, or git rebase --abort
              DetachedHead        check out <env>
              UnknownBoundary     give --cut for the bottom link; an upper link is rebased with git itself
              CutNotAncestor      check the --cut ref
              MissingRef          check the named ref
              InvertedOrder       list the chain top of stack first, BASE last, each environment once
  --dry-run that this is a plan with no change, then per link "<env> onto <predecessor>" and a table of REPO,
            BOUNDARY, and SOURCE, "-" for a skipped repo
```

### Help

```text
# product:declare-stack-order; the topic crates/core/topics/ws/restack.md, referenced from the ws group topic
states  ENVs are listed top of stack first and BASE last; each ENV is replayed onto the one after it, the last onto
        BASE, and the bottom link runs first
        a repository participates in a link where both its env branch and its predecessor resolve; pinned projects
        never do; --cut sets the bottom link's boundary only; --dry-run shows each link and its repositories
        a conflict is resolved with git in its worktree, then the same command is run again
```
