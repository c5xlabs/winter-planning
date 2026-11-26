# Push

The `ws push` command: publishing each selected worktree to the branch its own tracking names on `origin`, and keeping
the workspace's local copy of that branch level when it was level before. Its options and refusals are the `ws push` row
of [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); shape A is
that of [output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), and
this leaf owns the document. Selection is [target-patterns.md](./target-patterns.md) with `EnvRule::Present`; every
`git push` is a remote call of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), Remote
calls, whose `resolve` decides a held pin. The drift notice is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`; "WPS" is
`modules/workspace/workspace_push_service.py`, "WRR" `modules/workspace/internal/write_repo_repository.py`, "WH"
`modules/workspace/handlers/workspace_handler.py`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/push.rs — product:push-worktree-destinations,
// decision:parallel-workspaces/coordinated-git/remote-policy, decision:parallel-workspaces/coordinated-git/pinned-skips
pub(crate) enum PinnedScope { Exclude, Include, Only }   // default; --include-pinned; --only-pinned

/// The status of one row: this epic's key on the reference's outcome, under pinned-skips.
pub(crate) enum PushStatus {
    Pushed,          // "pushed"          git push ran and succeeded
    UpToDate,        // "up_to_date"      nothing ahead; git push did not run
    Failed,          // "failed"          git push, or a read before it, failed
    NoUpstream,      // "no_upstream"     no origin destination to push to
    SkippedPinned,   // "skipped_pinned"  a pinned worktree with commits, left out by the pinned scope
    HeldPin,         // "held_pin"        a standalone held at a tag or commit
}
pub(crate) struct LocalFastForward { pub branch: String, pub advanced: bool, pub commits: u64,
    pub skipped_reason: Option<String> }   // "not in sync" | "dirty" | "ff failed"
pub(crate) struct PushOutcome { pub repo_name: String, pub pushed: bool, pub commits: u64,
    pub error: Option<String>, pub local_ff: Option<LocalFastForward>, pub status: PushStatus }
pub(crate) struct EnvPush { pub env: String, pub repos: Vec<PushOutcome> }
pub(crate) struct EnvSkipped { pub env: String, pub reason: String }
pub(crate) struct PushReport { pub envs: Vec<EnvPush>, pub standalone: Vec<PushOutcome>, pub skipped: Vec<EnvSkipped> }

pub(crate) fn push(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path,
    patterns: &[Pattern], scope: Scope, pinned: PinnedScope) -> Result<PushReport, PushError>;
pub(crate) enum PushError { Config(Box<ConfigError>), Selection(SelectionError), Internal(Box<InternalError>) }

// termination.rs — GitOperation gains Push ("push"), TrackingRead ("tracking read"), TrackingAheadProbe
// ("tracking-ahead probe"), LocalFastForward ("local fast-forward")
```

## Interfaces

### Order

```text
# reference: WH:778-795, WPS:49-116
1. root discovery and the load; 2. the drift notice
3. selection with PATTERNS, default */*, unless --standalone; --standalone and --all reach every standalone
4. one target at a time (fan-out.md, serial): each reached environment in order, its selected worktrees in
   order, then each standalone in merged order
5. the document, or the text; exit 0 on success, else 1 (Termination::Reported) after it is written
success: no row is failed or no_upstream, and skipped is empty
```

### Per environment

```text
# reference: WPS:72-110,118-130
in scope    the selected worktrees the pinned scope keeps (Exclude: non-pinned; Include: all; Only: pinned)
ahead       each in-scope worktree is probed as status.md's worktree probe gives it:
              tracking_ahead  the branch.ab ahead count of its porcelain status; 0 with no resolving upstream
              main_ahead      the ahead count against origin/<main_branch>
            pinned:     a candidate when tracking_ahead > 0
            non-pinned: a candidate when tracking_ahead > 0 or main_ahead > 0
            a probe failure -> the row is failed with the rendered RepoError
no in-scope worktree, with Exclude:
            the pinned selected worktrees that would be candidates; when any, skipped gains
            {env, reason "<n> pinned repo(s) with commits skipped — use --include-pinned or --only-pinned"}, and
            the environment has no EnvPush. Otherwise the environment is left out.
            # as the reference: an env skip fails the run
in-scope worktrees present:
            one EnvPush, its rows in selection order:
              a candidate                 the push below
              not a candidate             up_to_date, pushed false, commits 0
              # registered deviation, pinned-skips: the reference leaves a non-candidate out of the document
            with Exclude, each pinned selected worktree that would be a candidate adds a skipped_pinned row,
            pushed false, after the in-scope rows, and does not fail the run
            # registered deviation, pinned-skips: the reference names skipped pinned commits only when no
            # non-pinned worktree was in scope
```

### Pushing a worktree

```text
# reference: WPS:132-195, WRR:587-599,606-744, modules/workspace/internal/branch_tracking.py:10-54
destination(wt)    the branch it tracks on origin, read from config only, so a never-pushed branch resolves:
                     h := git symbolic-ref --quiet --short HEAD; exit 1 -> None
                     git config --get branch.<h>.remote, git config --get branch.<h>.merge
                       either exit 1 -> None; other non-zero -> RepoError GitFailure { TrackingRead, repo }
                     remote is not exactly "origin", or merge does not start "refs/heads/" -> None
                     -> merge less "refs/heads/"
non-pinned         B := destination(wt); None -> no_upstream, error "no upstream — run `winter ws connect` first"
                   commits := git rev-list --count origin/<B>..HEAD when git rev-parse --verify --quiet origin/<B>
                              exits 0, else git rev-list --count origin/<main_branch>..HEAD; a failure counts 0
                              with a warning
                   git push -u origin HEAD:refs/heads/<B>
pinned             T := tracking_operand of its active branch (refresh.md, Domain), no commit check
                   commits := git rev-list --count <T>..HEAD, or 0 with no T
                   git push origin
                   # decision:parallel-workspaces/coordinated-git/remote-policy: origin is the only remote
landing            L := destination(wt), for pinned worktrees too; pre := git rev-parse --verify --quiet origin/<L>
                   before the push, None when it exits non-zero
push failure       failed, pushed false, error the rendered RepoError, logged at warning; no local fast-forward
push success       pushed, commits as counted; then the local fast-forward when L and pre are both set
```

```text
# local fast-forward, in <root>/projects/<repo>: product:push-worktree-destinations
local   := git rev-parse --verify --quiet refs/heads/<L>; non-zero -> None (no local copy)
local != pre                          -> {L, advanced false, skipped_reason "not in sync"}
new     := git rev-parse --verify --quiet refs/remotes/origin/<L>; non-zero or new == local -> None
holder  := the worktree path of the `worktree <path>` record whose `branch refs/heads/<L>` line follows it in
           git worktree list --porcelain; none -> git update-ref refs/heads/<L> <new> <local>
           -> {L, advanced true, commits git rev-list --count <local>..<new>}
holder dirty (git::dirty)             -> {L, advanced false, skipped_reason "dirty"}
otherwise, in holder: git merge --ff-only origin/<L> -> {L, advanced true, commits <local>..HEAD}
any git failure here                  -> logged at warning; {L, advanced false, skipped_reason "ff failed"}; the
                                         row stays pushed
```

### Standalones

```text
# reference: WPS:111-114,197-201, WRR:793-818,1073-1093
per standalone, in merged order:
  declares a ref resolving as Tag or Commit (resolve of standalone-pins.md)
                          -> held_pin, pushed false, error null; no push, and the run does not fail
                          # registered deviation, pinned-skips: the reference finds no upstream on the detached
                          # HEAD and fails the run
  a ref that does not resolve -> failed, error the rendered resolve error
  T := tracking_operand of its active branch (refresh.md), no commit check; none
                          -> no_upstream, error "no upstream — set one with `git branch --set-upstream-to`"
  ahead := git rev-list --count <T>..HEAD; non-zero exit -> failed, RepoError GitFailure { TrackingAheadProbe }
  ahead == 0              -> up_to_date (registered deviation, pinned-skips: the reference leaves it out)
  git push origin         -> pushed, commits ahead; failure -> failed
```

### Result

```text
--json, shape A
{"envs": [{"env": <name>, "repos": [outcome...]}], "standalone": [outcome...], "skipped": [{"env", "reason"}],
 "missing": [entry...]}       # missing: target-patterns.md, Missing targets in results
outcome: {"repo_name", "pushed": bool, "commits": int, "error": string | null,
          "local_ff": null | {"branch", "advanced", "commits", "skipped_reason": string | null},
          "status": "pushed" | "up_to_date" | "failed" | "no_upstream" | "skipped_pinned" | "held_pin"}
status is this epic's key; every other key and its value is the reference's (WH:1130-1132, json_render.py:11-26)
text (WH:1134-1196)
  no row pushed, failed, or no_upstream, and no skip -> "No repos with commits to push", dim
  sectioned when more than one environment, or one and standalones in scope: the env name, bold, per section
  per section a table REPO | PUSHED | COMMITS [| LOCAL]: PUSHED is yes, failed, up to date, no upstream,
  skipped (pinned), or held; COMMITS the count, or the error; LOCAL only when some row has local_ff:
  "<branch> +<n>" or "<branch> skipped (<reason>)"
  each skipped env: "! <env>: <reason>"
  standalones in scope with no row: "No standalone repos to push", dim
  last: "✓ push complete" on success, else "! push had errors or skipped envs"
```
