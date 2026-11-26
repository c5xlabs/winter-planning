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
pub(crate) struct Chain { pub envs: Vec<String>, pub base: String, pub cut: Option<String> }  // envs top of stack first
pub(crate) enum Refusal {
    MissingRef,               // "refused-missing-ref"
    Dirty,                    // "refused-dirty"
    RebaseInProgress,         // "refused-rebase-in-progress"
    DetachedHead,             // "refused-detached-head"
    InvertedOrder,            // "refused-inverted-order"
    UnknownBoundary,          // "refused-unknown-boundary"
    CutNotAncestor,           // "refused-cut-not-ancestor"
}
pub(crate) struct PlanRefusal {
    pub result: Refusal, pub env: String, pub predecessor: String,
    pub repos: Vec<String>,           // sorted; empty for a chain-shape refusal
    pub element: Option<String>,      // MissingRef: the env, BASE as typed, or "--cut"; InvertedOrder: the repeated cut
}
pub(crate) enum Source { ForkPoint, Cut }              // "fork-point" | "cut"
pub(crate) struct LinkRepo { pub repo_name: String, pub boundary: Option<Sha>, pub source: Option<Source> }
pub(crate) struct Link { pub env: String, pub predecessor: String, pub repos: Vec<LinkRepo> }
pub(crate) enum Plan { Refused(Vec<PlanRefusal>), Links(Vec<Link>) }  // Links bottom link first
pub(crate) fn plan(ports: &Ports<'_>, workspace: &Workspace, chain: &Chain) -> Result<Plan, Termination>;
```

## Interfaces

### Reads

```text
# reference: modules/workspace/env_restack_plan_service.py:60-200, internal/read_repo_repository.py:676-757
links         (envs[i], envs[i+1]) for each i, the last (envs[n-1], BASE): argument order, top first
repos         every non-pinned project, in configuration order; no environment lookup and no registry read:
              the worktree is <root>/<env>/<repo> whether or not it exists
pred_ref      the bottom link: expand(BASE) per repo; every other link: refs/heads/<predecessor>
env_ref       refs/heads/<env>                     # engineering:stack-boundary-integrity: a tag never stands in
tip(wt, ref)  git rev-parse --verify --quiet <ref> -> the sha, or None on any non-zero exit, a missing worktree, or
              a directory that is no repository
participates  env tip and predecessor tip both Some; otherwise the repo is skipped in that link
ancestor(a,b) git merge-base --is-ancestor <a> <b>: exit 0 -> true; any other exit, or a missing worktree -> false
fork(wt)      git merge-base --fork-point <pred_ref> <env_ref>: exit 0 with non-empty stdout -> the sha; else None
rebasing(wt)  git rev-parse --git-path rebase-merge, then rebase-apply: either path exists -> true
dirty(wt)     git status --porcelain --untracked-files=no (git::dirty): untracked files never count
branch(wt)    git symbolic-ref --quiet HEAD stripped of refs/heads/; exit 1 -> detached
a git that cannot be started is InternalError; dirty and branch reading a repo error end the run with it, exit 1
```

### Refusals

```text
# all-or-nothing: any refusal and no link is built; refusals are listed in the order below
1. repeated      the first value of envs ++ [BASE] that occurs twice: the first link whose env is that value,
                 InvertedOrder, repos [], element None; else --cut equal to an env or BASE: the bottom link,
                 InvertedOrder, element the cut. A repeat is the only refusal reported; nothing is read
2. reads         every link, every repo
3. missing refs  each link, top first, whose env tip is None in every repo -> MissingRef, element the env
                 the bottom link, when some repo has its env tip and every such repo lacks the predecessor tip
                 -> MissingRef, element BASE as typed
4. per link, top first, over its participating repos:
   safety        Dirty, then RebaseInProgress, then DetachedHead (HEAD not on refs/heads/<env>), each one refusal
                 naming its repos
   boundaries    the bottom link with --cut: per repo, cut_tip := tip(expand(cut)); None -> MissingRef with element
                 "--cut"; ancestor(cut_tip, env tip) (applies) or ancestor(predecessor tip, env tip) (already done)
                 -> boundary cut_tip, Cut; neither -> CutNotAncestor; MissingRef before CutNotAncestor
                 every other link: fork(wt); None -> UnknownBoundary naming the repos; else boundary, ForkPoint
                 # no merge-base fallback; --cut is never a boundary for an upper link
   inversion     with any boundary, and no repo whose env tip equals its boundary: when in every repo with a
                 boundary the env tip differs from the predecessor tip and ancestor(env tip, predecessor tip)
                 -> InvertedOrder naming those repos
                 # declared limit: a chain typed backwards in any other shape is not detected (product:declare-stack-order)
```

### Plan

```text
links         reversed: the bottom link first, the order of execution
link repos    every non-pinned repo in configuration order; a skipped one carries boundary and source None
frozen        boundaries are read once, here; execution reads new bases afresh (restack-execution.md)
```

### Result

```text
--json, one document, no discriminator field (output-formats.md)
  refused   {"refusals": [{"result", "env", "predecessor", "repos": [...], "element": str|null}...]}; exit 1
  --dry-run {"links": [{"env", "predecessor", "repos": [{"repo_name", "boundary": sha|null, "source":
            "fork-point"|"cut"|null}...]}...]}; exit 0, no write call at all
text
  refused   that the restack was refused with no change, then per refusal its result, "<env> onto <predecessor>",
            its repos, and its element when there is one
  --dry-run that this is a plan with no change, then per link "<env> onto <predecessor>" and a table of REPO,
            BOUNDARY, and SOURCE, "-" for a skipped repo
```
