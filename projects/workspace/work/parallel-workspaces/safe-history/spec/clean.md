# Clean

The `ws clean` command: removing untracked files and directories from each selected worktree, never ignored ones. Its
options and combination rules are the `ws clean` rows of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); its NDJSON
events, whose fields this leaf owns, are the `ws clean` vocabulary of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md); the prompt is
the confirmation of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md),
Confirmation, under the `ws clean` row of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Confirmation prompts. Selection is that of [target-patterns.md](../../coordinated-git/spec/target-patterns.md), and the
command is serial, as [fan-out.md](../../coordinated-git/spec/fan-out.md) lists. The drift warning is that of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md). This is not the
top-level `clean` of `epic:running-environments/resource-provisioning`. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/history/clean.rs — product:remove-untracked-content, engineering:clean-partial-accounting
// Git's clean, below: one call each; a failed call, a worktree that is no repository, and an output the strict parse
// cannot place are each a RepoError naming GitOperation::Clean, carried as GitFailure::Repo
pub fn list(ports: &Ports<'_>, wt: &Selected) -> Result<Vec<String>, GitFailure>;   // git clean -nd, strict parse
pub fn remove(ports: &Ports<'_>, wt: &Selected) -> Removed;                         // git clean -fd
pub struct Removed { pub paths: Vec<String>, pub failure: Option<GitFailure> }      // paths: those already gone
pub enum Parse { Strict, Lenient }
pub fn parse(output: &str, prefix: &str, parse: Parse) -> Result<Vec<String>, String>;   // Err: the diagnosis

pub(crate) struct RepoClean { pub env: String, pub repo: String, pub paths: Vec<String> }  // count = paths.len()
pub(crate) struct CleanFailure {
    pub env: String, pub repo: String,
    pub message: String,                  // the repo error's message and git's stderr, or the parse diagnosis
    pub unattempted: Vec<(String, String)>,   // registered deviation: (env, repo) of every target not reached
}
pub(crate) struct CleanReport {
    pub dry_run: bool,
    pub missing: Vec<(Selected, Missing)>,   // target-patterns.md, Missing targets in results
    pub repos: Vec<RepoClean>,               // empty: nothing matched; every count 0: nothing to clean
    pub failure: Option<CleanFailure>,
}
pub(crate) fn preview(ports: &Ports<'_>, targets: &[Selected]) -> Result<CleanReport, CleanError>;
pub(crate) fn clean(ports: &Ports<'_>, targets: &[Selected], dry_run: bool) -> CleanReport;
// The command file's one domain call (context/architecture/changes.md, A command): Running, steps 1-10, from the start
// path, as the built locations::list and pins::repin take it. confirm is called with the preview exactly when step 7
// prompts; the command file builds it over confirm.rs (runtime.md, Confirmation), rendering the preview and asking
// "Continue?". Its Err is the ending, answered as Stopped with nothing removed
pub(crate) fn run(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[Pattern],
    force: bool, dry_run: bool, confirm: &mut dyn FnMut(&CleanReport) -> Result<(), Termination>)
    -> Result<CleanEnd, CleanError>;
pub(crate) enum CleanEnd { Reported(CleanReport), Stopped(Termination) }
// the failures before anything is removed: the load, the selection (target-patterns.md, SelectError), and a list
// call of preview (Running, step 5). The command file maps: Config -> Termination::Config, EnvName ->
// Termination::Command(CommandError::EnvName), Selection -> Termination::Command(CommandError::Selection), Repo ->
// Termination::Repo, Internal -> Termination::Internal, each box as it is
pub enum CleanError {
    Config(Box<ConfigError>), EnvName(EnvNameError), Selection(SelectionError), Repo(Box<RepoError>),
    Internal(Box<InternalError>),
}
// crates/core/src/history.rs — the module's own error; termination.rs gains CommandError::History(HistoryError)
pub enum HistoryError {
    JsonNeedsForce,                       // ws clean --json without --force or --dry-run; exit 1, nothing read
}
// crates/core/src/termination.rs — RepoMessage gains, each with GitOperation::Clean
NotAGitWorktree { path },              // "<path> is not a git worktree"
UnrecognisedClean { path, line },      // "git clean output not recognised at <path>: <line>"; the strict parse's diagnosis
```

Visibility: `list`, `remove`, `parse`, `Removed`, `Parse`, and `CleanError` are `pub` inside their private module and
re-exported by name from `lib.rs`, each module's names on a `pub use` line of their own, with every type their
signatures name (`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands
each tests it before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an
`allow` (`context/standards/rust.md`); `HistoryError` is re-exported with them (changes.md, An error). `parse` is
re-exported as `parse_clean`, since `targets::parse` holds `parse` at the crate root. `Selected` is `pub` by
[target-patterns.md](../../coordinated-git/spec/target-patterns.md), Domain. Everything else here stays `pub(crate)`:
`run`, `CleanEnd`, `preview`, `clean`, and the report types land with the command that calls them.

## Interfaces

### Git's clean

```text
# reference: modules/workspace/internal/write_repo_repository.py:39-75,439-525
list(wt)    git clean -nd     LC_ALL=C, LANGUAGE=C, LC_MESSAGES=C; prefix "Would remove "
remove(wt)  git clean -fd     same environment; prefix "Removing "
            never -x or -X: ignored files stay; -f is git's clean.requireForce, not Winter's --force
            a nested repository is not removed and not listed by -nd; an empty directory is both
parse       each line, trimmed, that is non-empty; a line starting with the prefix yields the rest as one path, kept
            exactly as git prints it, quoting included, and a directory with its trailing "/"
  strict    no path, and some line that starts with none of "Would skip ", "Skipping ", "warning:"
            -> the output is unrecognised: the parse fails with that diagnosis
  exit 0    strict parse; list's failure is the RepoError of the call
  exit != 0 remove only: a lenient parse of stdout, every recognised path kept: those are already gone
            # engineering:clean-partial-accounting: git clean deletes what it can and exits non-zero
a worktree path that is no repository -> the repo error NotAGitWorktree, "<path> is not a git worktree"
the strict parse's failure -> the repo error UnrecognisedClean, naming the first line it could not place
```

### Running `ws clean PATTERNS...`

```text
# reference: modules/workspace/handlers/workspace_handler.py:561-673, env_clean_service.py:58-110
order
  0. the command file: the combination rules of command-surface.md; --json without --force or --dry-run ->
     HistoryError::JsonNeedsForce, exit 1, nothing read
  steps 1-10 are run's
  1. root discovery and the load; a failure is CleanError::Config, exit 1
  2. the drift warning
  3. targets := the non-pinned worktrees PATTERNS select (target-patterns.md), in its order; a refusal is
     CleanError::EnvName or Selection, exit 1
  4. no targets -> Nothing matched (Result); exit 0
  5. preview := list(wt) for each target, in order; a failure ends the command with its RepoError, exit 1, before
     anything is removed
  6. preview total 0 -> Nothing to clean (Result); exit 0, no prompt
  7. neither --force nor --dry-run -> confirm(preview): its closure renders the preview (every path, per worktree, and
     that this cannot be undone), then confirm("Continue?"), even for one worktree; declined or end of input ->
     Aborted, exit 1, nothing removed
  8. --dry-run -> list(wt) again per target, nothing removed; otherwise remove(wt) per target, in order, every target
     visited, also one the preview showed empty
     # decision:parallel-workspaces/destructive-approval: the files are listed again at removal, so a file created
     # after the prompt is removed and reported; the report, not the preview, is the record
  9. a target whose call fails, or whose strict parse fails -> its RepoClean holds the paths recognised, the failure
     names it, and the run stops; the later targets are untouched (decision:parallel-workspaces/partial-execution)
     # registered deviation: the reference ends an unrecognised success output with a bare repo error, losing the
     # outcomes of the earlier targets, and does not name the unattempted ones
 10. exit 0 without a failure, else 1 (Reported)
```

### Result

```text
--json  NDJSON, each line flushed (shape C)
  {"type": "clean_started", "dry_run": bool}     target_missing events precede it (target-patterns.md, Missing
                                                 targets in results)
  {"type": "repo_clean", "env", "repo", "count", "paths"}       one per target reached, in order, empty ones too;
                                                                paths is what git reported removed, or would remove
  {"type": "clean_completed", "success": bool, "total": <sum of count>}
      plus "failure": {"env", "repo", "message", "unattempted": [{"env", "repo"}...]} when it stopped
      # unattempted: registered deviation
  nothing matched: started with dry_run, then completed with success true and total 0
  nothing to clean: started, one repo_clean per target with count 0, completed
text    nothing matched: one line naming PATTERNS; nothing to clean: one line
  --dry-run: per worktree with paths, "<env>/<repo> (<count>)" and each path; the totals; that nothing was removed
  removed:   the same listing of what went, then the count of paths and of worktrees touched
  stopped:   additionally the worktree it stopped on with its message, that the paths listed are gone for good, and
             the worktrees not reached
```
