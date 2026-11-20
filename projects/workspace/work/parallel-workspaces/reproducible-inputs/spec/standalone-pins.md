# Standalone pins

How a standalone's `ref` resolves, how its lock entry is materialized, advanced, re-pinned, and frozen, and what the
commands report. The lock file format, its errors, the `Set a standalone ref` writer, and write integrity are those of
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md) and
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md);
this leaf builds them. Every git call goes through the program port of
[child-processes.md](../../repository-catalog/spec/child-processes.md), with the child environment it states. Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`. Message
wording is the binary's own unless a line below quotes it (`decision:functional-parity`).

## Domain

```rust
// engineering:typed-outcomes
pub enum RefKind { Branch, Tag, Commit }               // the lock's `kind` values: "branch", "tag", "commit"

pub struct Resolution { pub kind: RefKind, pub commit: Sha }   // Sha: 40 lowercase hex characters
// For an annotated tag, `commit` is the tag object's id, as `rev-parse --verify refs/tags/<t>` answers it: the lock
// records that id, it never equals HEAD, and the up-to-date check and head drift read it as moved

pub struct LockEntry { pub name: String, pub r#ref: String, pub kind: RefKind, pub commit: Sha }

// crates/core/src/config.rs — the StandaloneEntry of repository-addition.md, Domain, gains its declared ref:
pub struct StandaloneEntry { pub name: String, pub url: Option<String>, pub path: Option<String>,
                                    pub r#ref: Option<String> }   // None when absent; the load's refusals are unchanged
// A standalone's checkout location is the catalog's: location-listing.md, Interfaces, standalone.

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
    fn integrate(&self, path: &Path, target: &str, mode: PullMode, autostash: bool) -> Result<Integration, RepoError>;
}

// What `advance` answers: a pin outcome of its own, or the pull's integration passed through
pub enum Advanced { Pin(PinOutcome), Integrated(Integration) }
```

`RepoError` is that of [location-listing.md](../../repository-catalog/spec/location-listing.md): a git call's
failure, carrying its exit. A run succeeds unless an outcome is `refused` or `pin_error`, or a pull's integration is
`Diverged` (`modules/workspace/models/service_model.py:514-522`). A `pin_ref` written `<rendered>` below is the
`RepoError` as Python's `str()` renders it (`modules/workspace/models/domain_model.py:322-337`): the message, then
`  $ <program> <subcommand> <args>`, `  cwd: <cwd>`, `  exit <n>` (with ` (killed by signal <s>)` for a negative
exit), and `  stderr: <stderr, trimmed>`, each on its own line and only when present.

Every type and entry the slices re-export for tests is `pub` inside its private module, and so is every type their
signatures name, the program port's `Call`, `Finished`, `Exit`, `Program`, and `RunPrograms` included; `lib.rs`
re-exports them by name (changes.md, A domain module).

```rust
// engineering:typed-outcomes; the RepoMessage of results-and-exit-codes.md, Domain, gains the causes the reference
// raises as RepoError; the wording is the reference's, <name> and <ref> quoted as Python's repr quotes them
UnresolvableRef { r#ref: String, path: PathBuf }
    // "unresolvable ref <ref> at <path>: not a branch, tag, or commit SHA"; cwd the path
DirtyRePin { name: String }
    // "refusing to re-pin <name>: working tree has uncommitted changes; commit or stash first (or run
    // `winter ws fetch <name>` to sync refs)"
NotPinned { name: String }          // "standalone repo <name> has no `ref` configured — nothing to update"
NoPinnedNamed { name: String }      // "no pinned standalone repo named <name>"
NoStandaloneNamed { name: String }  // "no standalone repo named <name>"
// the last four carry no detail field: the reference passes no cwd or an empty one

// crates/core/src/pins.rs — the module's own error sum, which materialize, advance, the re-pin run, and the freeze
// run answer; the command maps it: Repo -> Termination::Repo, Config -> Termination::Config, exit 1 either way
pub enum PinError { Repo(RepoError), Config(ConfigError) }   // Config holds the lock's ConfigError::Lock and
                                                                    // the Set a standalone ref writer's errors
```

## Interfaces

### Git operations

```text
# crates/core/src/git.rs; engineering:domain-io-boundaries; every call through the program port, `git` run in the
# checkout as its cwd
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
# the bound child-processes.md leaves to the epics that fetch; the program port's Call gains `timeout: Option<Duration>`
timeout     WINTER_GIT_TIMEOUT_S of the snapshot, float seconds; unset -> 40
            unparseable, inf, or nan -> warning "ignoring invalid WINTER_GIT_TIMEOUT_S=<raw> (expected float seconds);
            using default 40s", 40; <= 0 -> warning "ignoring non-positive timeout <value> (would kill every git
            subprocess immediately); using default 40s", 40; read on every call
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
            standards/rust.md, Output and environment access, names it as the exception to "reads no clock"; the retry
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
resolve(path, ref) -> Result<Resolution, RepoError>
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

Exit 1 when the run does not succeed; in text mode a run with no outcome exits 0. Before selection, in both modes, the
command writes the drift notice of [drift-warning.md](../../repository-catalog/spec/drift-warning.md)
(`modules/workspace/handlers/workspace_handler.py:723`).

## Flows

### Materialize a pin

```text
# product:recorded-input-materialization; called by the standalone reconciliation of epic:parallel-workspaces/workspace-bootstrap
# after its clone or fetch; reference: modules/workspace/init_service.py:485-560
materialize(repo, path) -> Result<Option<(RefKind, Sha)>, PinError>
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
# product:standalone-reference-policy; called by the standalone integration of epic:parallel-workspaces/coordinated-git
# after its fetch, for a standalone that declares a ref; reference: modules/workspace/workspace_sync_service.py:426-504
advance(repo, mode, autostash, integration: &dyn IntegrateCheckout) -> Result<Advanced, PinError>
  r := resolve(path, repo.ref)          # an error propagates, as PinError::Repo, to the pull's per-repository handling
  Tag | Commit -> Pin(HeldPin), pin_ref repo.ref                  # never moves, whatever the mode
  Branch:
    dirty := not clean(path)
    dirty and not autostash -> Pin(PinError) "branch pin <name> has uncommitted changes; commit/stash or pass --autostash"
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
  a selected checkout missing on disk -> stderr "warning: standalone/<name> — missing on disk (run `winter ws init`)",
                                         dropped
emit pull_started
per selected standalone, in order (an outcome does not stop the next):
  fetch(path); its failure is logged at warning and resolution continues against the refs on disk
                                        # a moved remote tag is not fetched over the local one, so it reads up to date
  dirty := not clean(path)
  dirty and not autostash -> PinError "refusing to re-pin <name>: uncommitted changes; commit/stash or pass --autostash"
  dirty -> stash_push; its failure -> PinError <rendered>
  r := resolve(path, ref); its failure -> PinError <rendered>
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
  path missing                         -> Refused "<name> is not cloned (run `winter ws init`)"
  c := head(path); its failure         -> Refused <message>               # read before the dirty check; the message
                                                                          # alone (workspace_sync_service.py:647)
  not force and not clean(path)        -> Refused "<name> has uncommitted changes, so its commit does not describe it;
                                          commit/stash or pass --force"
  set `ref = "<c>"` in config.toml, else config.local.toml, on the first block whose effective name (`name`, else
  derived from `url`) is the standalone's
                                       # the Set a standalone ref writer, through write integrity; neither file
                                       # declares it -> Refused "<name> is not declared in config.toml or config.local.toml"
  upsert {name, ref: c, kind: Commit, commit: c} -> Pinned, pin_ref c[..8]
                                       # a LockError here ends the whole run as PinError::Config, exit 1, with the
                                       # ref already written to the config file and no pull_completed
                                       # (modules/workspace/workspace_sync_service.py:671-677)
emit repo_synced per outcome, then pull_completed
# a commit that exists only locally is recorded without comment
```

### Pin snapshot

```text
# product:standalone-reference-policy; read by the workspace status of epic:parallel-workspaces/coordinated-git
# reference: modules/workspace/workspace_snapshot_service.py:557-601,
#            modules/workspace/handlers/workspace_handler.py:1512-1523
snapshot(standalones) -> Vec<PinSnapshot>
  standalones with a ref, in merged order; none -> empty, and the lock is not read
  the lock is read once; a LockError is logged at warning and read as an empty lock: a read-only view, the one place
  a malformed lock is read as absent (modules/workspace/workspace_snapshot_service.py:570-574)
  head(path) per standalone; its failure is logged at debug and leaves head_commit None
```
