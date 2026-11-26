# Refresh

The `ws fetch` and `ws pull` commands: refreshing each selected project once through its source checkout, and pulling
each selected worktree onto its own upstream. Their options and refusals are the `ws fetch` and `ws pull` rows of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); shape C is
that of [output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), whose
event fields this leaf owns. Selection is [target-patterns.md](./target-patterns.md) with `EnvRule::Present`; the
parallelism is [fan-out.md](./fan-out.md); the integration is [integration.md](./integration.md); a standalone that
declares a `ref` advances through `advance` of
[standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md), whose
`fetch` and Remote calls every `git fetch` below uses. The drift notice is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`; "WSS" is
`modules/workspace/workspace_sync_service.py`, "WRR" `modules/workspace/internal/write_repo_repository.py`, "WH"
`modules/workspace/handlers/workspace_handler.py`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/refresh.rs — product:integrate-selected-repositories,
// decision:parallel-workspaces/coordinated-git/fetch-freshness
pub(crate) enum Scope { Project, Standalone, All }        // default; --standalone; --all; also ws merge's, ws push's

/// The tracking operand of the active branch at `path`, with no commit check: "<R>/<M less a leading refs/heads/>"
/// of branch.<head>.remote R and branch.<head>.merge M; None when HEAD is detached or either is unset (ws pull,
/// step 4). ws pull adds the commit check; ws push and ws status read it as it is.
pub fn tracking_operand(ports: &Ports<'_>, path: &Path) -> Result<Option<String>, GitFailure>;

/// The refresh of one project through its source checkout, <root>/projects/<repo>.
pub(crate) struct SourceSync {
    pub commits: u64,                  // how far the source checkout's main moved; 0 when skipped or level
    pub warning: Option<String>,       // the checkout is not on its main branch: fetched, not fast-forwarded
}
pub(crate) fn source_sync(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, repo: &str,
    source: &Path, main_branch: &str) -> Result<SourceSync, GitFailure>;

pub(crate) enum FetchEvent {
    Started,                                                                         // fetch_started
    Fetched { scope: Label, repo: String, result: Result<SourceSync, Box<RepoError>> }, // repo_fetched
    Completed { success: bool },                                                     // fetch_completed
}
pub(crate) enum Label { Project, Standalone }              // "project" | "standalone"

pub(crate) enum PullEvent {
    Started,                                                                         // pull_started
    RefreshFailed { scope: Label, repo: String, error: Box<RepoError> },             // refresh_failed
    SourceWarning { repo: String, warning: String },                                 // source_checkout_warning
    Synced(Synced),                                                                  // repo_synced
    Completed { success: bool },                                                     // pull_completed
}
pub(crate) struct Synced {
    pub scope: String,                 // the env name, or "standalone"
    pub repo: String,
    pub result: SyncResult,
    pub upstream: Option<String>,      // the ref integrated, as integrated; None when nothing was integrated
    pub commits: u64, pub ahead: u64, pub behind: u64,
    pub pin_ref: String,               // as standalone-pins.md gives it; "" otherwise
}
/// Integrated of integration.md, plus the pull's own: no_upstream, error, and PinResult's held_pin, re_pinned,
/// pin_error passed through from advance.
pub(crate) enum SyncResult { /* string values in the event table below */ }

pub(crate) fn fetch(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path,
    patterns: &[Pattern], scope: Scope, sink: &mut dyn FnMut(&FetchEvent) -> Result<(), Termination>)
    -> Result<RunEnd, RefreshError>;
pub(crate) fn pull(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path,
    patterns: &[Pattern], scope: Scope, fallback: Fallback, autostash: bool,
    sink: &mut dyn FnMut(&PullEvent) -> Result<(), Termination>) -> Result<RunEnd, RefreshError>;
// RunEnd is that of standalone-pins.md: Completed { success } or Stopped(Termination); Completed { success: true }
// with no Started also stands for "nothing to do", which the command file tells by the events it saw
pub(crate) enum RefreshError { Config(Box<ConfigError>), Selection(SelectionError), Internal(Box<InternalError>) }

// termination.rs — GitOperation gains SourceSync ("source checkout sync")
```

Visibility: `tracking_operand` is `pub` inside its private module and re-exported by name from `lib.rs` on a `pub use`
line of the module's own (`context/architecture/changes.md`, A domain module, of `winter-workspace`), since it lands
with `ws fetch`, which does not call it, so its slice tests it before `ws pull`, `ws push`, and `ws status` do
(`context/standards/testing.md`) and the gate finds nothing unused without an `allow` (`context/standards/rust.md`).
Everything else here stays `pub(crate)`.

## Interfaces

### Source sync

```text
# reference: WRR:165-208, modules/workspace/models/service_model.py:461-490
source_sync(repo, source, main)
  1. fetch(source)                                  git fetch origin; a failure is its RepoError: the project's
                                                    refresh failed
  2. git symbolic-ref --quiet --short HEAD          exit 1 -> detached
  3. not on main (detached, or a branch other than main):
       -> SourceSync { commits 0, warning "source checkout is on <branch 'b' | a detached HEAD>, not main_branch
          '<main>'; fetched origin but did not fast-forward" }
  4. h0 := git rev-parse HEAD
     git merge --ff-only origin/<main>              non-zero -> RepoError GitFailure { SourceSync, repo }: the
                                                    project's refresh failed, though step 1 updated its refs
     commits := git rev-list --count <h0>..<HEAD after>; a failure is a warning and counts 0
feature worktrees are never touched by a refresh
```

### `ws fetch`

```text
# reference: WH:675-693, WSS:92-195
order
  1. root discovery and the load; 2. the drift notice
  3. selection with PATTERNS, default */*, unless --standalone; --standalone and --all reach every standalone
  4. projects := each project with a selected worktree, once, in order of its first selected worktree; pinned
     worktrees count like any other. Missing worktrees were warned and dropped by the selection
  5. no project and no standalone -> no event; text "Nothing to fetch", dim; --json writes nothing; exit 0
  6. fetch_started; then, as fan-out.md REMOTE_TASKS, one group per project (source_sync) and one per standalone
     (fetch(path), nothing fast-forwarded); repo_fetched as each finishes
  7. fetch_completed { success: no repo_fetched failed }; exit 0 on success, else 1 (Termination::Reported)
```

### `ws pull`

```text
# reference: WH:695-720, WSS:197-504; product:integrate-selected-repositories
order
  1-3 as ws fetch
  4. targets := each selected worktree, upstream decided at its turn:
       pinned      origin/<main_branch>
       non-pinned  its own upstream, read after the refresh by tracking_operand:
                     h := git symbolic-ref --quiet --short HEAD; exit 1 -> no upstream
                     R := git config --get branch.<h>.remote, M := git config --get branch.<h>.merge; either
                     exit 1 -> no upstream
                     operand := "<R>/<M less a leading refs/heads/>"; then the commit check: it must name a
                     commit (git rev-parse --verify --quiet <operand>^{commit}), else no upstream
                     # a configured but unfetched upstream, and a local upstream R = ".", read as no upstream
                   no upstream -> Synced no_upstream, nothing integrated
  5. no target and no standalone -> no event; text "Nothing to pull", dim; --json writes nothing; exit 0
  6. pull_started; then, as fan-out.md REMOTE_TASKS:
       per project group: source_sync once; a failure -> refresh_failed, and the worktrees still integrate on the
         refs already local; a warning -> source_checkout_warning; then each target in selection order:
         integrate(path, upstream, fallback, autostash) -> repo_synced
       per standalone: fetch(path); a failure -> refresh_failed, and the standalone still integrates on local refs
         no ref declared: tracking_operand of its active branch, without the commit check; no branch or no
                          R and M -> no_upstream; else integrate, where an operand naming no commit ends
                          missing_ref (WRR:762-773 integrates it unchecked and reports diverged)
         ref declared:    advance(...) of standalone-pins.md with this leaf's integrate as IntegrateCheckout;
                          Pin(o) -> its PinResult; Integrated(i) -> i; a PinError -> result "error"
       a GitFailure::Repo inside a target's step -> that target's repo_synced with result "error", and its
       rendered error as a stderr line; the other targets go on (decision:parallel-workspaces/partial-execution)
  7. pull_completed { success }; exit 0 on success, else 1
success: no refresh_failed; no worktree result in diverged, missing_ref, dirty_blocked, autostash_failed,
abort_failed, error; no standalone result in those or no_upstream, pin_error, refused
# registered deviations, decision:parallel-workspaces/coordinated-git/fetch-freshness and integration-conflicts:
# the reference logs a failed refresh as a warning and succeeds, and reports every new result as diverged
```

### Events

```json
{"type": "target_missing", "env": "<env> | standalone", "repo": "<name>", "reason": "absent | not_a_repository"}
{"type": "fetch_started"}
{"type": "repo_fetched", "scope": "project", "repo": "<name>", "success": true, "commits": 0, "error": null, "warning": null}
{"type": "fetch_completed", "success": true}
{"type": "pull_started"}
{"type": "refresh_failed", "scope": "project", "repo": "<name>", "error": "<rendered RepoError>"}
{"type": "source_checkout_warning", "repo": "<name>", "warning": "<text>"}
{"type": "repo_synced", "scope": "<env> | standalone", "repo": "<name>", "result": "<SyncResult>", "commits": 0, "ahead": 0, "behind": 0, "pin_ref": ""}
{"type": "pull_completed", "success": true}
```

```text
repo_fetched   success false -> error is the rendered RepoError, commits 0, warning null; a warning is set only
               on success; a standalone's commits is always 0
refresh_failed this epic's event, under fetch-freshness; never written by ws update
target_missing ws fetch and ws pull, before their started event, as target-patterns.md, Missing targets in results,
               gives; never written by ws update
repo_synced    result: up_to_date, fast_forwarded, merged, rebased, diverged, no_upstream, held_pin, re_pinned,
               pin_error, and this epic's missing_ref, dirty_blocked, autostash_failed, abort_failed, error
               commits: Integrated's commits, else 0; ahead, behind: Diverged's, else 0
env_skipped    in the vocabulary of output-formats.md and never written by ws pull: the reference builds no env skip
output-formats.md's ws pull row lacks source_checkout_warning, which the reference writes
(modules/workspace/pull_reporter.py:136-137); refresh_failed joins it after it
```

Text, one line per event (`modules/workspace/fetch_reporter.py:28-58`, `modules/workspace/pull_reporter.py:28-92`):

| Event or result                                                             | Stream                                         | Line                                                           |
| --------------------------------------------------------------------------- | ---------------------------------------------- | -------------------------------------------------------------- |
| `fetch_started`, `pull_started`                                             | stdout                                         | `→ fetching`, `→ pulling`                                      |
| `repo_fetched` failed                                                       | stderr                                         | `[<scope>/<repo>] error: <error>`                              |
| `repo_fetched` with a warning                                               | stderr                                         | `[<scope>/<repo>] warning: <warning>`                          |
| `repo_fetched` ok                                                           | stdout                                         | `[<scope>/<repo>] ok (+<commits>)` or `up to date` when 0      |
| `refresh_failed`                                                            | stderr                                         | `[<scope>/<repo>] refresh failed: <error>`                     |
| `source_checkout_warning`                                                   | stderr                                         | `[project/<repo>] warning: <warning>`                          |
| `diverged`                                                                  | stderr                                         | `[<scope>/<repo>] diverged: +<ahead>/-<behind>`                |
| `missing_ref`, `dirty_blocked`, `autostash_failed`, `abort_failed`, `error` | stderr                                         | `[<scope>/<repo>] <what happened>`                             |
| `no_upstream`                                                               | stdout for a worktree, stderr for a standalone | `[<scope>/<repo>] no upstream`                                 |
| `up_to_date`, `fast_forwarded`, `merged`, `rebased`                         | stdout                                         | `[<scope>/<repo>] <result> (+<commits>)`                       |
| `held_pin`, `re_pinned`, `pin_error`                                        | as `ws update`'s table in standalone-pins.md   | `held @ <pin_ref>` for `held_pin`                              |
| `*_completed` true                                                          | stdout                                         | a blank line, then `✓ fetch complete` or `✓ pull complete`     |
| `*_completed` false                                                         | stderr                                         | a blank line, then `✗ fetch had errors` or `✗ pull had errors` |

After a pull that did not succeed, in text mode under the default ff-only, when any result is `diverged`, the hint
"retry with --merge or --rebase, or resolve with raw git", dim, on stdout (WH:712-716). The sink is built as
`ws update`'s in standalone-pins.md, Running `ws update`: stdout rows through `Results::line`, stderr rows through
`diagnostics::notice`.
