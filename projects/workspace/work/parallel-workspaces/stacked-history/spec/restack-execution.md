# Restack execution

Running an unrefused plan of `ws restack`: replaying each link base-ward first, stopping at the first conflict or
failure, and the result a re-run continues from. The plan, its reads, and the JSON of a refusal and a dry run are those
of [restack-plan.md](./restack-plan.md). Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
`winter-workspace` at `127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/restack/execute.rs — product:replay-dependent-work, product:resume-stack-rewrite,
// decision:parallel-workspaces/stacked-history/resume-policy
pub(crate) enum Outcome { UpToDate, Rebased }           // "up-to-date" | "rebased"
pub(crate) struct Completed { pub env: String, pub repo_name: String, pub result: Outcome }
pub(crate) struct Conflict {
    pub env: String, pub predecessor: String, pub repo_name: String,
    pub replayed_commit: Sha, pub conflicted_paths: Vec<String>,
}
pub(crate) struct Failure { pub env: String, pub repo_name: String, pub message: String }
pub(crate) struct Report {
    pub completed: Vec<Completed>,
    pub stop: Option<Stop>,           // None: the whole plan ran
    pub remaining: Vec<Link>,         // from the link the run stopped on, to the top; empty when complete
}
pub(crate) enum Stop { Conflict(Conflict), Failure(Failure) }
pub(crate) fn execute(ports: &Ports<'_>, workspace: &Workspace, links: &[Link]) -> Report;
// no state is written anywhere for a later run: a re-run plans from Git alone
```

## Interfaces

### Replay

```text
# reference: modules/workspace/env_restack_service.py:30-178, internal/write_repo_repository.py:383-437
per link, in plan order (bottom first); per participating repo, in configuration order:
  pred_ref, env_ref, tip, ancestor   as restack-plan.md, Reads, read now, not at planning
  env_tip := tip(env_ref); newbase := tip(pred_ref); oldbase := the frozen boundary
    either tip None -> Failure naming the ref that no longer resolves
    # the reference's assertion error is an artifact of Python under decision:functional-parity, not a deviation
  cut_exempt := source is Cut and ancestor(oldbase, env_tip)      # the cut range is still to be dropped
  not cut_exempt and ancestor(newbase, env_tip) -> UpToDate; no write
  otherwise  git rebase --onto <newbase> <oldbase> <env>   (checks out <env> as a side effect)
    exit 0 -> Rebased, also when nothing past oldbase replays and the branch fast-forwards
    non-zero, classified by the rebase state git left (engineering:stack-stop-classification):
      the first of git rev-parse --git-path rebase-merge/stopped-sha, rebase-apply/original-commit that exists
        -> Conflict: replayed_commit its content, trimmed; conflicted_paths the lines of
           git diff --name-only --diff-filter=U; the repo is left mid-rebase, never aborted
      neither -> Failure with the repo error's message and git's stderr
a Conflict or a Failure stops the run: later repos of that link and every upper link are not touched
remaining := the links from the stopped one to the top, plan-shaped; no rollback, nothing pushed, tracking unchanged
```

### Resume

```text
# decision:parallel-workspaces/stacked-history/resume-policy
the engineer resolves with git (git rebase --continue or --abort) in that worktree, then runs the same chain, BASE, and
--cut again; the re-run plans afresh: a worktree still mid-rebase is refused RebaseInProgress (restack-plan.md), a
completed link reads UpToDate, and a cut repo already replayed has its env tip off the cut range, so cut_exempt is
false and it reads UpToDate; a re-run without --cut plans by fork point instead, as the reference does
```

### Result

```text
--json, one document, written at the end (shape A)
  {"links": [{"env", "predecessor", "repos": [{"repo_name", "boundary", "source", "outcome"}...]}...],
   "completed": [{"env", "repo_name", "result"}...],
   "conflict": {"env", "predecessor", "repo_name", "replayed_commit", "conflicted_paths": [...]} | null,
   "failure": {"env", "repo_name", "message"} | null,
   "remaining": {"links": [plan-shaped link...], "refusals": []}}
  outcome   "up-to-date" | "rebased" | "conflict" for a repo reached; "skipped" for a non-participating repo;
            null for a participating repo never reached
  links     every link of the plan, bottom first
exit        0 when the whole plan ran; 1 after a conflict or a failure (Reported)
text        per link "<env> onto <predecessor>" and a table of REPO, BOUNDARY, SOURCE, and OUTCOME ("-" for none);
            then the conflict (the replayed commit, the link, the repo, each conflicted path, and to resolve with
            git rebase --continue and re-run the same command with the same --cut), or the failure with its repo
            and message, or that the restack is complete
restack never pushes; force-pushing a rebased branch is the engineer's (epic:parallel-workspaces/coordinated-git)
```
