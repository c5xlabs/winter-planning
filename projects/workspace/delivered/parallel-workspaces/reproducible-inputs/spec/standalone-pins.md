# Standalone pins

How a standalone's `ref` resolves, how its lock entry is materialized, advanced, re-pinned, and frozen, and what the
commands report. The lock file format, its errors, the `Set a standalone ref` writer, and write integrity are those of
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md) and
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md);
this leaf builds them. Every git call goes through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md), with the
child environment it states. The `ws update` arguments, the bare-name refusal, and the usage errors are those of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md); shape C is
that of [output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), JSON
shapes; `RepoMessage` and `Termination` are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain; `StandaloneRepository` is that of
[discovery-and-loading.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/discovery-and-loading.md),
Domain. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at
`b57b020`. Message wording is the binary's own unless a line below quotes it (`decision:functional-parity`).

## Domain

```rust
// engineering:typed-outcomes
pub enum RefKind { Branch, Tag, Commit }               // the lock's `kind` values: "branch", "tag", "commit"

// crates/core/src/git.rs — a commit or object id as git prints it, 40 lowercase hex characters from git itself; a lock
// commit is kept as read, unvalidated, as the reference keeps str() of it
pub type Sha = String;

pub struct Resolution { pub kind: RefKind, pub commit: Sha }
// For an annotated tag, `commit` is the tag object's id, as `rev-parse --verify refs/tags/<t>` answers it: the lock
// records that id, it never equals HEAD, and the up-to-date check and head drift read it as moved

pub struct LockEntry { pub name: String, pub r#ref: String, pub kind: RefKind, pub commit: Sha }
// A field of a lock entry that is not a string is read as its text, as Python's str() gives it, and is not refused:
// `commit = 42` reads as the commit "42", as the recorded scenario ws-update/lock-field-not-string shows. Such a
// commit equals no resolved one, so the next re-pin moves the checkout and rewrites the entry with the resolved
// commit as a string. Reference: modules/workspace/internal/config_lock_repository.py:69-72

// A standalone is the loaded StandaloneRepository of discovery-and-loading.md, Domain, unchanged: below, repo.name is
// its common.name, repo.ref its reference, and its checkout the catalog's location (location-listing.md, Interfaces,
// standalone). It is pinned when reference is Some, Some("") included, as the reference tests `is None`.

// crates/core/src/config_lock.rs — the lock's reader and upsert, LockError beside them; the file's path is LOCK_FILE,
// ".winter/config.lock", beside CONFIG_FILE in crates/core/src/workspace_files.rs

pub enum PinResult {                                   // the `result` value of a repo_synced event
    UpToDate,                                          // "up_to_date"
    RePinned,                                          // "re_pinned"      pin_ref: the new commit, first 8 characters
    HeldPin,                                           // "held_pin"       pin_ref: the declared ref
    Pinned,                                            // "pinned"         pin_ref: the frozen commit, first 8 characters
    AlreadyPinned,                                     // "already_pinned" pin_ref: the declared ref
    Refused,                                           // "refused"        pin_ref: the reason
    PinError,                                          // "pin_error"      pin_ref: the reason
}

pub struct PinOutcome { pub repo: String, pub result: PinResult, pub pin_ref: String }

pub struct PinSnapshot {                               // one per standalone that declares a `ref`
    pub name: String,
    pub r#ref: String,
    pub kind: Option<RefKind>,                         // from the lock entry; None without one
    pub locked_commit: Option<Sha>,
    pub config_ref_drift: bool,                        // an entry exists and its ref differs from the declared ref
    pub head_drift: bool,                              // an entry exists, HEAD was read, and HEAD differs from its commit
    pub head_commit: Option<Sha>,                      // None when HEAD cannot be read
}

// The pull's integration of one checkout onto a target ref, which the pull of epic:parallel-workspaces/coordinated-git
// implements; reference: modules/workspace/internal/write_repo_repository.py:804-856
pub enum PullMode { FfOnly, Merge, Rebase }
pub enum Integration {
    UpToDate,
    FastForwarded { commits: u32 },                    // commits: HEAD..target before the integration
    Merged { commits: u32 },                           // Merge, after a fast-forward was refused
    Rebased { commits: u32 },                          // Rebase, after a fast-forward was refused
    Diverged { ahead: u32, behind: u32 },
}
pub trait IntegrateCheckout {
    fn integrate(&self, path: &Path, target: &str, mode: PullMode, autostash: bool) -> Result<Integration, GitFailure>;
}

// What `advance` answers: a pin outcome of its own, or the pull's integration passed through
pub enum Advanced { Pin(PinOutcome), Integrated(Integration) }
```

`RepoError` is that of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md): a git
call's failure, carrying its exit. A run succeeds unless an outcome is `refused` or `pin_error`, or a pull's integration
is `Diverged` (`modules/workspace/models/service_model.py:514-522`). A `pin_ref` written `<rendered>` below is the
`RepoError` as Python's `str()` renders it (`modules/workspace/models/domain_model.py:322-337`): the message, then
`$ <program> <subcommand> <args>`, `cwd: <cwd>`, `exit <n>` (with `(killed by signal <s>)` for a negative exit), and
`stderr: <stderr, trimmed>`, each on its own line, indented by two spaces, and only when present; lines are joined by
`\n` with none after the last. `PinResult`, `PinOutcome`, and that rendering, `pins::outcome::rendered`, live in
`crates/core/src/pins/outcome.rs`.

Every type and entry the slices re-export for tests is `pub` inside its private module, and so is every type their
signatures name or their tests build: the file port's `ReadFiles`, `WriteFiles`, and `OsFiles`; the program port's
`Call`, `Finished`, `Exit`, `Program`, `RunPrograms`, and `OsPrograms` with `OsPrograms::new`; `CommandTracer`;
`Diagnostics` with the `Log` and `Level` it holds and `Log::DEFAULT`; `LockError` and `LockEntryFault`; `Sha`,
`GitFailure`, `Ports`, and `PinError`. `lib.rs` re-exports them by name (changes.md, A domain module), each module's
names on a `pub use` line of its own.

Every warning, notice, and log record below is written by the domain entry through the `Diagnostics` of
[repository-list.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/repository-list.md), Domain,
which each entry takes beside its ports; a command file writes none
(`decision:parallel-workspaces/workspace-configuration/domain-reporting`). What the reference logs at warning or debug
is `diagnostics::record` at that level; a line quoted with its `warning:` prefix, which the reference echoes, is
`diagnostics::notice`.

A placeholder written `<x!r>` in quoted wording is the value as Python's `repr` quotes it; one written `<x>` is the
value as it is.

```rust
// engineering:typed-outcomes; the RepoMessage of results-and-exit-codes.md, Domain, gains the causes the reference
// raises as RepoError; the wording is the reference's (gitpython_repository.py:241, init_service.py:539-540,
// workspace_sync_service.py:557,561,624)
UnresolvableRef { r#ref: String, path: PathBuf }
    // "unresolvable ref <ref!r> at <path>: not a branch, tag, or commit SHA"; the RepoError's cwd is the path
DirtyRePin { name: String }
    // "refusing to re-pin <name!r>: working tree has uncommitted changes; commit or stash first (or run
    // `winter ws fetch <name>` to sync refs)"
NotPinned { name: String }          // "standalone repo <name!r> has no `ref` configured — nothing to update"
NoPinnedNamed { name: String }      // "no pinned standalone repo named <name!r>"
NoStandaloneNamed { name: String }  // "no standalone repo named <name!r>"
// a RepoError built from any of the last four carries no cwd: the reference passes none or an empty one

// crates/core/src/git.rs — what a git operation answers on failure: the call's RepoError, or a git that could not be
// started, which child-processes.md ends as InternalError::Unexpected; boxed as Termination boxes them
pub enum GitFailure { Repo(Box<RepoError>), Internal(Box<InternalError>) }

// crates/core/src/pins.rs, whose flows are its pub(crate) submodules
// pins/{outcome,resolve,materialize,advance,repin,freeze,snapshot}.rs. The ports every pin entry takes, as one value:
pub struct Ports<'p> {
    pub files: &'p dyn ReadFiles, pub writer: &'p dyn WriteFiles,
    pub programs: &'p dyn RunPrograms, pub tracer: &'p dyn CommandTracer,
}
// The module's own error sum, which materialize, advance, snapshot, the re-pin run, and the freeze run answer; the
// command maps it: Repo -> Termination::Repo, Config -> Termination::Config, Internal -> Termination::Internal
pub enum PinError {
    Repo(Box<RepoError>),
    Config(Box<ConfigError>),     // the lock's ConfigError::Lock and the Set a standalone ref writer's errors
    Internal(Box<InternalError>), // a git that could not be started
}
```

## Interfaces

### Git operations

```text
# crates/core/src/git.rs; engineering:domain-io-boundaries; functions over &dyn RunPrograms and &dyn CommandTracer,
# which a command file hands its domain entry as cx.programs and cx.tracer; Context gains nothing; every call through
# the program port, `git` run in the checkout as its cwd
read
  head(path)            rev-parse HEAD                                      fails: not a repository, no HEAD
  clean(path)           status --porcelain                                  clean iff empty output; any failure reads dirty;
                                                                            untracked files count as dirty
  verify(path, rev)     rev-parse --verify <rev>                            a non-zero exit is no match, not an error
write
  fetch(path)           fetch origin                                        bounded: Remote calls, below; an existing
                                                                            local tag is kept when its remote tag moved
  checkout_branch(p, b) checkout -B <b> --track origin/<b>                  resets local <b> to origin/<b>: local-only
                                                                            commits on <b> are left to the reflog
  checkout_detached(p,c) checkout --detach <c>
  stash_push(path)      stash push                                          untracked files are not stashed; a tree
                                                                            dirty only by them stashes nothing and the
                                                                            later pop takes whatever stash is on top
  stash_pop(path)       stash pop
```

Reference: `modules/workspace/internal/gitpython_repository.py:205-375`,
`modules/workspace/internal/write_repo_repository.py:734-743`.

### Remote calls

```text
# reference: modules/workspace/internal/git_ops_service.py:24-81,113-149,165-247
# the bound child-processes.md leaves to the epics that fetch; the program port's Call gains `timeout: Option<Duration>`,
# None on every call but fetch's; fetch takes the invocation's ProcessEnv and the entry's Diagnostics
timeout     WINTER_GIT_TIMEOUT_S of the snapshot, float seconds; unset -> 40
            unparseable as Python's float() parses (surrounding blanks and `_` between digits allowed), inf, or nan
            -> warning "ignoring invalid WINTER_GIT_TIMEOUT_S=<raw!r> (expected float seconds); using default 40s",
            40; <= 0, -inf included -> warning "ignoring non-positive timeout <value> (would kill every git
            subprocess immediately); using default 40s", 40, <value> as Python's float repr gives it (-1.0, 0.0,
            -inf); read on every call
            # Python's float() accepts inf and nan and passes them to GitPython; Rust's Duration refuses both, so
            # they take the invalid-value path: a difference of the libraries, not a change of function
wait        OsPrograms, in programs.rs, starts no thread: it polls the child's stdout and stderr with poll(2) until
            both close or the deadline on the monotonic clock passes, reading each as it is ready, so a child that
            writes more than a pipe buffer to either stream is never stalled; through the rustix crate (event and
            process features), which the library gains
expiry      as GitPython's kill_process (git/cmd.py:1306-1328): the direct children of git are listed with
            `ps --ppid <pid>`, run by OsPrograms itself without a timeout, git is sent SIGKILL, then each child; a
            listing or kill that fails is ignored; the streams are then read to their end; the exit is Signal(9),
            and stderr is replaced, as git/cmd.py:1338-1342 does, by `Timeout: the command "git <args joined by
            spaces>" did not complete in <timeout, truncated to whole seconds> secs.`
attempts    at most 3; a failure is retried when it is transient and attempts remain, else it is the call's RepoError
transient   a signal exit, or stderr matching, case-insensitively, any of:
              Connection closed by .* port 22 | fatal: the remote end hung up unexpectedly |
              kex_exchange_identification | Connection timed out | Timeout: the command .* did not complete
delay       base = min(1 * 2^(attempt-1), 8); delay = base + base * 0.5 * u, u uniform in [-1, 1], clamped to [0, 8]
            seconds; slept before the next attempt; u is drawn through the fastrand crate, which the library gains
clock       programs.rs is the one library file that reads the monotonic clock, sleeps, and draws a random number:
            context/standards/rust.md, Output and environment access, names it as the exception to "reads no clock"; the retry
            loop takes the sleep and the draw from it, so a test fixes both
notice      warning per retry: "transient git error (attempt <n>/3): <stderr, trimmed> — retrying in <delay, 2
            decimals>s"
```

### Operation spans

```text
# reference: modules/workspace/internal/git_operation.py:14-86; engineering:observability-isolation
# the tracer port of child-processes.md; attributes and failure marking as location-listing.md, tracing
one span per operation, named "git <operation>", covering every git call the operation makes, retries included;
resolve opens its span itself and its verifies open none:
  head, verify via resolve     "git rev-parse"   one span for resolve's up to three verifies
  clean                        "git status"
  fetch                        "git fetch"
  checkout_branch, _detached   "git checkout"
  stash_push, stash_pop        "git stash push", "git stash pop"
attribute winter.repo = the standalone's resolved name; never an argument, a path, or output
```

### Resolution

```text
# decision:parallel-workspaces/reproducible-inputs/resolution-and-locks
# reference: modules/workspace/internal/gitpython_repository.py:222-243
resolve(ports, path, ref) -> Result<Resolution, GitFailure>
  verify refs/remotes/origin/<ref>   -> Branch
  else verify refs/tags/<ref>        -> Tag
  else verify <ref>^{commit}         -> Commit
  else RepoError UnresolvableRef
# first match wins; a shadowed candidate is not reported; no network
```

### `ws update` events and text

Event vocabulary of shape C, one line per event (`modules/workspace/pull_reporter.py:95-140`):

```json
{"type": "pull_started"}
{"type": "repo_synced", "scope": "standalone", "repo": "<name>", "result": "<PinResult>", "commits": 0, "ahead": 0, "behind": 0, "pin_ref": "<detail>"}
{"type": "pull_completed", "success": true}
```

Text, one line per event (`modules/workspace/pull_reporter.py:28-92`,
`modules/workspace/handlers/workspace_handler.py:722-751`):

| Event or result         | Stream | Line                                             |
| ----------------------- | ------ | ------------------------------------------------ |
| `pull_started`          | stdout | `→ pulling`                                      |
| `up_to_date`            | stdout | `[standalone/<name>] up to date`                 |
| `re_pinned`             | stdout | `[standalone/<name>] re-pinned → <pin_ref>`      |
| `pinned`                | stdout | `[standalone/<name>] pinned → <pin_ref>`         |
| `already_pinned`        | stdout | `[standalone/<name>] already pinned @ <pin_ref>` |
| `refused`               | stderr | `[standalone/<name>] refused: <pin_ref>`         |
| `pin_error`             | stderr | `[standalone/<name>] pin error: <pin_ref>`       |
| `pull_completed`, true  | stdout | a blank line, then `✓ pull complete`             |
| `pull_completed`, false | stderr | a blank line, then `✗ pull had errors`           |
| no outcome at all       | stdout | `Nothing to update`, dim, after `pull_completed` |

Dim is SGR 2 around the line when `cx.out_is_terminal`, and the plain line otherwise, as
[runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Terminal, colour, width,
gives styling.

Exit 1 when the run does not succeed; in text mode a run with no outcome exits 0. Before selection, in both modes, the
re-pin and freeze entries each write the drift notice of
[drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md)
(`modules/workspace/handlers/workspace_handler.py:723`).

### Running `ws update`

```rust
// crates/core/src/pins/repin.rs and pins/freeze.rs — the domain entries the ws update command file calls
pub enum PinEvent { Started, Synced(PinOutcome), Completed { success: bool } }   // pull_started, repo_synced,
                                                                                    // pull_completed
// The command's sink, called once per event as it is reached, with the entry's Diagnostics so a text-mode stderr row
// can be written; an Err stops the run at once, after any pending stash_pop, and the entry answers RunEnd::Stopped.
pub type EventSink<'s> = dyn FnMut(&PinEvent, &mut Diagnostics<'_>) -> Result<(), Termination> + 's;

// How a run that raised no PinError ended; the command maps Completed { success: true } -> Termination::Success,
// Completed { success: false } -> Termination::Reported (exit 1, the rows already written), Stopped(t) -> t
pub enum RunEnd { Completed { success: bool }, Stopped(Termination) }

pub fn repin(ports: Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[String],
    autostash: bool, sink: &mut EventSink<'_>) -> Result<RunEnd, PinError>;
pub fn freeze(ports: Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path, patterns: &[String], force: bool,
    sink: &mut EventSink<'_>) -> Result<RunEnd, PinError>;
```

```text
the sink, built by the command file over output::Results of runtime.md, Writing more than one piece of a result,
which this command is the first caller of and builds:
  --json   each event is Results::event of its shape C line
  text     a stdout row is Results::line; a stderr row (refused, pin_error, pull_completed false, each with its
           blank line where the table gives one) is diagnostics::notice; `Nothing to update` follows pull_completed
           when no Synced event was seen
the sink's stderr rows are the one exception to "a command file writes no notice": they are the command's own
result rows, which the reference's reporter writes to stderr, and not a notice of the domain
```

## Flows

### Materialize a pin

```text
# product:parallel-workspaces/workspace-bootstrap/recorded-input-materialization; called by the standalone reconciliation of epic:parallel-workspaces/workspace-bootstrap
# after its clone or fetch; reference: modules/workspace/init_service.py:485-560
materialize(ports, root, repo) -> Result<Option<(RefKind, Sha)>, PinError>   # path: the catalog's location of repo
  repo.ref is None                      -> None
  entry := lock[repo.name]              # LockError propagates as PinError::Config
  entry present and entry.ref == repo.ref:
    Branch        -> checkout_branch(path, entry.ref)        # lands on origin/<ref> as last fetched
    Tag | Commit  -> checkout_detached(path, entry.commit)
    -> Some(entry.kind, entry.commit)                         # the recorded commit, also for a branch; lock untouched
  otherwise:
    not clean(path) -> RepoError DirtyRePin
    r := resolve(path, repo.ref)
    Branch -> checkout_branch(path, repo.ref); else checkout_detached(path, r.commit)
    upsert {name, ref: repo.ref, kind: r.kind, commit: r.commit}   # after the checkout; a failed one writes nothing
    -> Some(r.kind, r.commit)
# no dirty guard on the fresh path: git's own checkout refusal is the only guard there
```

### Advance a pin on pull

```text
# product:parallel-workspaces/coordinated-git/integrate-selected-repositories; called by the standalone integration of epic:parallel-workspaces/coordinated-git
# after its fetch, for a standalone that declares a ref; reference: modules/workspace/workspace_sync_service.py:426-504
advance(ports, diagnostics, root, repo, mode, autostash, integration: &dyn IntegrateCheckout) -> Result<Advanced, PinError>
  r := resolve(path, repo.ref)          # an error propagates, as PinError::Repo or Internal, to the pull's handling
  Tag | Commit -> Pin(HeldPin), pin_ref repo.ref                  # never moves, whatever the mode
  Branch:
    dirty := not clean(path)
    dirty and not autostash -> Pin(PinError) "branch pin <name!r> has uncommitted changes; commit/stash or pass --autostash"
    dirty -> stash_push; its failure -> Pin(PinError) <rendered>
    i := integration.integrate(path, "origin/<ref>", mode, autostash)
    dirty -> stash_pop; its failure is logged at warning, naming the checkout, and changes no outcome
                                        # an Err from integrate propagates without the pop
    FastForwarded -> upsert {name, ref, Branch, head(path)}; Pin(RePinned), pin_ref head[..8], commits 0
                                        # a failed head read is PinError::Repo, a LockError PinError::Config
    UpToDate | Merged | Rebased | Diverged -> Integrated(i), the lock untouched  # a merge or rebase moves the checkout, not the record
```

### Re-pin (`ws update`)

```text
# product:update-standalone-inputs, decision:parallel-workspaces/reproducible-inputs/update-preservation
# reference: modules/workspace/workspace_sync_service.py:506-588,680-787
select(patterns):
  pinned := standalones with a ref, in merged order
  a pattern is a glob when it holds `*`, `?`, or `[`, else a literal (modules/workspace/pattern_match.py:9-11,86-101);
  a glob matches a name as fnmatchcase does: case-sensitive, `*`, `?`, `[seq]`, `[!seq]`
  each literal pattern:
    names a pinned standalone   -> kept
    names an unpinned standalone -> RepoError NotPinned, exit 1
    names none                  -> RepoError NoPinnedNamed, exit 1
  no pattern -> every pinned standalone; else the pinned standalones any pattern matches; a glob matching none adds none
  a selected checkout missing on disk -> notice "warning: standalone/<name> — missing on disk (run `winter ws init`)",
                                         dropped
emit pull_started
per selected standalone, in order (an outcome does not stop the next):
  fetch(path); its failure is logged at warning and resolution continues against the refs on disk
                                        # a moved remote tag is not fetched over the local one, so it reads up to date
  dirty := not clean(path)
  dirty and not autostash -> PinError "refusing to re-pin <name!r>: uncommitted changes; commit/stash or pass --autostash"
  dirty -> stash_push; its failure -> PinError <rendered>
  r := resolve(path, ref); its failure -> warning record "resolve_ref failed for standalone <name>: <rendered>",
                                        then PinError <rendered> (workspace_sync_service.py:726)
  r.commit == head(path) and lock[name].commit == r.commit -> UpToDate     # no checkout, no lock write
  Branch -> checkout_branch(path, ref); else checkout_detached(path, r.commit); its failure -> PinError <rendered>,
                                        and the lock is not written
  upsert {name, ref, r.kind, r.commit}  -> RePinned, pin_ref r.commit[..8]
  finally, when stashed: stash_pop; its failure is logged at warning, naming the checkout, and changes no outcome
  a failed head read or LockError here ends the whole run as that PinError, exit 1, after the pop; no pull_completed
emit repo_synced per outcome as it is reached, then pull_completed
```

### Freeze (`ws update --freeze`)

```text
# product:freeze-standalone-inputs; no network, no checkout change
# reference: modules/workspace/workspace_sync_service.py:590-678, config/internal/write_winter_configuration_repository.py:56-70,115-145
select(patterns):
  each literal pattern naming no standalone -> RepoError NoStandaloneNamed, exit 1   # globs as the re-pin's select
  no pattern -> every standalone; else every standalone any pattern matches, pinned or not
emit pull_started
per selected standalone, in order:
  declares a ref                       -> AlreadyPinned, pin_ref the ref
  path missing                         -> Refused "<name!r> is not cloned (run `winter ws init`)"
  c := head(path); its failure         -> Refused <message>               # read before the dirty check; the message
                                                                          # alone (workspace_sync_service.py:661)
  not force and not clean(path)        -> Refused "<name!r> has uncommitted changes, so its commit does not describe it;
                                          commit/stash or pass --force"
  set `ref = "<c>"` in config.toml, else config.local.toml, on the first block whose effective name (`name`, else
  derived from `url`) is the standalone's, matched by the effective_name of config_edit.rs that the removal uses
                                       # the Set a standalone ref writer, through write integrity; neither file
                                       # declares it -> Refused "<name!r> is not declared in config.toml or config.local.toml"
  upsert {name, ref: c, kind: Commit, commit: c} -> Pinned, pin_ref c[..8]
                                       # a LockError here ends the whole run as PinError::Config, exit 1, with the
                                       # ref already written to the config file and no pull_completed
                                       # (modules/workspace/workspace_sync_service.py:671-677)
emit repo_synced per outcome, then pull_completed
# a commit that exists only locally is recorded without comment
```

### Pin snapshot

```text
# product:parallel-workspaces/coordinated-git/inspect-workspace-git; read by the workspace status of epic:parallel-workspaces/coordinated-git
# reference: modules/workspace/workspace_snapshot_service.py:557-601,
#            modules/workspace/handlers/workspace_handler.py:1512-1523
snapshot(standalones) -> Vec<PinSnapshot>
  standalones with a ref, in merged order; none -> empty, and the lock is not read
  the lock is read once; a LockError is logged at warning and read as an empty lock: a read-only view, the one place
  a malformed lock is read as absent (modules/workspace/workspace_snapshot_service.py:570-574)
  head(path) per standalone; its failure is logged at debug and leaves head_commit None
```

### Interrupts

An interrupt during `ws update` ends the invocation as the runtime's handler ends it for a command that does not defer
([runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Interrupts and signals): at
once, without a pop. A stash pushed before it stays in the checkout's stash list, and a lock write either landed whole
or not at all, through the delivered write integrity. The reference pops in a `finally` when Python raises its
interrupt; this binary does not defer for a stash, since changes.md defers only around a child process the command must
outlive.
