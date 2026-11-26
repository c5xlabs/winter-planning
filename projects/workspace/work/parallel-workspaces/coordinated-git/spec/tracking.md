# Tracking

The `ws connect` and `ws disconnect` commands: writing and removing each selected worktree's own upstream, with no fetch
and no check that the branch exists anywhere. Their arguments, the split of `FEATURE_BRANCH` off the last token, and the
refusals before the load are the `ws connect` and `ws disconnect` rows of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); shape A is
that of [output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), and
this leaf owns both documents. Selection is [target-patterns.md](./target-patterns.md) with `EnvRule::Named`; the drift
notice is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md); the push default
is [push-default.md](./push-default.md). Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
`winter-workspace` at `127cdf2`; "WH" is `modules/workspace/handlers/workspace_handler.py`. Message wording is the
binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/tracking.rs — product:worktree-upstream-control, decision:parallel-workspaces/coordinated-git/remote-policy
pub(crate) struct Changed { pub env: String, pub repo: String }
pub(crate) struct Stopped { pub at: Changed, pub error: Box<RepoError> }   // the worktree whose write failed

pub(crate) struct TrackingRun {
    pub changed: Vec<Changed>,        // connected, or disconnected, in selection order
    pub failed: Option<Stopped>,      // the first failure; the run stops there
    pub unattempted: Vec<Changed>,    // the selected worktrees after the failure, in selection order
}
pub(crate) fn connect(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[Pattern],
    feature_branch: &str) -> Result<TrackingRun, TrackingError>;
pub(crate) fn disconnect(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[Pattern])
    -> Result<TrackingRun, TrackingError>;
pub(crate) enum TrackingError { Config(Box<ConfigError>), Selection(SelectionError), Internal(Box<InternalError>) }

// termination.rs — GitOperation gains UpstreamProbe ("upstream probe"), UnsetUpstream ("upstream removal")
```

## Interfaces

### Order

```text
# reference: WH:314-409, modules/workspace/env_checkout_service.py:38-82
1. root discovery and the load; a failure is its ConfigError, exit 1
2. the drift notice
3. selection: EnvRule::Named over PATTERNS; an unknown literal env refuses here (target-patterns.md)
4. per selected worktree, in selection order, one at a time:
     pinned            skipped silently: never connected, never disconnected
     missing           warned and skipped by the selection
     otherwise         the writes below; a write that fails stops the run: that worktree is `failed`, every later
                       selected non-pinned worktree is `unattempted`, and nothing is rolled back
5. the result; exit 0 when nothing failed, else 1 (Termination::Reported) after the result is written
# registered deviation, decision:parallel-workspaces/partial-execution: the reference stops at the first failure
# with an unhandled git error, no document, and no list of what it already wrote
```

### Writes

```text
# reference: modules/workspace/internal/write_repo_repository.py:210-254,527-563,1054-1071
head(wt)        git symbolic-ref --quiet --short HEAD
                  exit 0 -> the branch, less its final newline
                  exit 1 -> detached: the worktree's env name, as the branch every feature worktree is made on
connect(wt, B)  h := head(wt); detached -> warning "<repo> is detached; writing branch.<h>.* without attaching
                HEAD: the worktree stays detached and untracked until something attaches it", and the writes go on
                git config branch.<h>.remote origin
                git config branch.<h>.merge refs/heads/<B>
                set_push_default_upstream(wt)
                B is written as given: no fetch, no ref check, no check that origin exists
                # decision:parallel-workspaces/coordinated-git/remote-policy: origin is the only remote
disconnect(wt)  h := head(wt)
                git config --get branch.<h>.remote
                  exit 1 -> no upstream: nothing written, and the worktree is still listed as disconnected
                  other non-zero -> RepoError GitFailure { UpstreamProbe, repo }
                git branch --unset-upstream <h>
                  non-zero -> RepoError GitFailure { UnsetUpstream, repo }
a git config write that fails -> RepoError GitFailure { Config, repo }
```

### Result

```text
--json, shape A
  ws connect     {"patterns": [<as typed>], "feature_branch": <B>, "connected": [{"env", "repo"}], "count": <n>,
                  "failed": null | {"env", "repo", "error": <rendered RepoError>}, "unattempted": [{"env", "repo"}]}
  ws disconnect  {"patterns": [...], "disconnected": [{"env", "repo"}], "count": <n>, "failed": ..., "unattempted": [...]}
  count is the length of connected or disconnected; failed and unattempted are this epic's additions and are
  null and [] on a run that did not stop
text
  nothing changed and nothing failed   "No worktrees matched: <patterns joined by spaces>", dim; exit 0
  changed                              "✓ Connected <n> worktree(s) → <B>" or "✓ Disconnected <n> worktree(s)",
                                       then "  <env>/<repo>" per changed worktree
  failed                               on stderr: "<env>/<repo>: <rendered RepoError>", then
                                       "not attempted: <env>/<repo>, ..." when any
<rendered RepoError> is pins::outcome::rendered of
../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md, Domain
```
