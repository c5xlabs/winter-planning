# Environment creation and reconciliation

`ws init <env>` and `ws init --all`: allocating an environment's index, creating its directory and one worktree per
declared project, and reapplying each worktree's identity, excludes, tracking, and setup commands; and, for `--all`,
finding every existing environment after the workspace part. Bare `ws init` and the workspace part of `--all` are
[reconciliation.md](../../../../delivered/parallel-workspaces/workspace-bootstrap/spec/reconciliation.md)'s, which
leaves TARGET and `--all` to this leaf (`decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope`); its
`InitEvent`, `Action`, `checkout::run` steps, set_config, exclude_path, Commands, and Events and text are used here as
they stand. The argument and options are the `ws init` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands, and
its refusals that file's `ws init` row of Combination rules, then [env-names.md](./env-names.md). Every git call goes
through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md); the index is
[allocation.md](./allocation.md)'s; the workspace exclude file is that of
[managed-content.md](../../../../delivered/parallel-workspaces/workspace-bootstrap/spec/managed-content.md); the fan-out
bound is `engineering:bounded-fan-out`'s; `push_default` and `set_push_default_upstream` are
[push-default.md](./push-default.md)'s. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
`winter-workspace` at `127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## System

```text
commands/ws/init.rs ── TARGET ──► env_create::init_env(ports, env, diagnostics, start, name, sink)
                   │              the load, the name check, the registry read, then reconcile_env
                   ├── --all ───► env_create::init_all(ports, env, diagnostics, start, sink)
                   │              the load, the registry read, bootstrap::reconcile (the workspace part), the
                   │              findings, discover, then reconcile_env per name
                   └── neither ─► bootstrap::reconcile                                  (reconciliation.md)
each branch is the command file's one domain call (context/architecture/changes.md, A command, of winter-workspace)
reconcile_env ─► state_registry::allocate ─► create_dir_all ─► workspace_exclude::write_block
              ─► per project, at most 4 at once: worktree::run ─► hooks on_env_init
```

Steps that other epics add, each after the step named:

| Step                                                          | After                       | Owner                                              |
| ------------------------------------------------------------- | --------------------------- | -------------------------------------------------- |
| Nested delegation and the child's `ws init`                   | the worktree's `cmd`        | `epic:parallel-workspaces/nested-workspaces`       |
| `on_env_init` hooks, run as invocation-contract.md, Hooks     | the projects fan-out        | `epic:agent-ready-operation/executable-extensions` |
| `on_workspace_reconcile` once, in the workspace part of --all | the workspace's standalones | `epic:agent-ready-operation/executable-extensions` |

invocation-contract.md is
[../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md](../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md);
the nested step is [delegation.md](../../nested-workspaces/spec/delegation.md).

## Domain

```rust
// crates/core/src/env_create.rs — product:environment-worktrees, product:environment-recovery
// engineering:lifecycle-step-isolation
// RunEnd is that of pins/repin.rs, already re-exported at the crate root, as reconciliation.md's reconcile answers it
// The command file's entries take the start path, as the built locations::list and pins::repin do, and do root
// discovery, the load, and the registry read themselves: Running ws init <env>, steps 1-4, and Running ws init --all
pub(crate) fn init_env(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path,
    name: &str, sink: &mut InitSink<'_>) -> Result<RunEnd, EnvInitError>;
pub(crate) fn init_all(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path,
    sink: &mut InitSink<'_>) -> Result<RunEnd, EnvInitError>;
// the failures that end the run before any target; the command file maps Config -> Termination::Config, EnvName ->
// Termination::Command(CommandError::EnvName), Repo -> Termination::Repo, Internal -> Termination::Internal, each box
// as it is. bootstrap::reconcile's InitError::Config and Internal are carried as Config and Internal
pub(crate) enum EnvInitError {
    Config(Box<ConfigError>), EnvName(EnvNameError), Repo(Box<RepoError>), Internal(Box<InternalError>),
}
pub(crate) fn reconcile_env(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>,
    workspace: &Workspace, name: &str, sink: &mut InitSink<'_>) -> RunEnd;
    // Completed { success } of the target; Stopped(the sink's ending) for a sink that cannot write
    // (reconciliation.md, Fan-out); it has no failure of its own: every other fault is an event of the target
pub(crate) fn discover(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, workspace: &Workspace)
    -> Result<Vec<String>, GitFailure>;

// crates/core/src/env_create/worktree.rs — one project's worktree in one env, steps in order
pub struct EnvWorktree<'r> {
    pub project: &'r ProjectRepository,
    pub source: PathBuf,              // <root>/projects/<name>
    pub location: PathBuf,            // <root>/<env>/<name>
    pub env: &'r str,                 // also the branch name
    pub inferred: Option<&'r str>,    // "<remote>/<branch>", from infer below
}
pub fn run(ports: &Ports<'_>, workspace: &Workspace, w: &EnvWorktree<'_>,
    sink: &mut dyn FnMut(InitEvent)) -> bool;
```

```rust
// crates/core/src/bootstrap.rs — Action gains; location <root>/<env>/<repo>
WorktreeCreated,      // "worktree_created"     detail ""                text "[<repo>] worktree created at <location>"
UpstreamInferred,     // "upstream_inferred"    detail the upstream      text "[<repo>] inferred upstream set: <detail>"
PinnedTrackingSet,    // "pinned_tracking_set"  detail the changes       text "[<repo>] pinned tracking set: <detail>"
WorktreeDetached,     // "worktree_detached"    detail ""                stderr "[<repo>] warning: HEAD is detached at
                      //                                                   <location>; not reattached"
WorktreeParked,       // "worktree_parked"      detail the branch        stderr "[<repo>] warning: <location> is on
                      //                                                   <detail>, not <env>; tracking left as it is"
                      //   both: registered deviation, decision:parallel-workspaces/environment-lifecycle/head-state

// crates/core/src/termination.rs — RepoMessage gains
SourceCheckoutMissing { name, path },   // "source checkout missing at <path>. Run `winter ws init` first."
NotAWorktreeOf { name, path },          // the location exists and is not a worktree of the project's source store
HeadDetached { name, path, upstream },  // "set-upstream-to <upstream> failed at <path>: HEAD is detached"
BadUpstream { name, upstream },         // "expected '<remote>/<branch>', got <upstream>"
// GitOperation gains BranchExists, WorktreeAdd, WorktreeList, CommonDir, Tracking, SetUpstream
```

Visibility: `EnvWorktree` and `run` of `env_create/worktree.rs`, and its tracking reads and write `head_branch`,
`setting`, `tracking`, and `set_upstream` (Git operations below), are `pub` inside their private module and re-exported
by name from `lib.rs`, each module's names on a `pub use` line of their own, with every type their signatures name
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands each tests it
before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`). `run` is re-exported as `reconcile_worktree`, since `process::run` holds that name at the
crate root, as `bootstrap::checkout::run` is re-exported as `run_checkout`. The git operations below are `pub` and
re-exported, as the built ones of `git.rs` are. Everything else here stays `pub(crate)`: `init_env` and `init_all` land
with the command that calls them.

## Interfaces

### Git operations

```text
# crates/core/src/git.rs gains; spans named "git <subcommand>", attribute winter.repo, as reconciliation.md
branch_exists(source, b)      show-ref --verify --quiet refs/heads/<b>     exit 0 true, 1 false, else GitFailure
worktree_add(source, p, b)    worktree add <p> <b>                         cwd source; an existing local branch
worktree_add_new(source, p, b, base)
                              worktree add <p> -b <b> --no-track <base>    cwd source; base = the project's
                                                                            main_branch, as a name: git resolves it
                                                                            to the local branch
                              # --no-track: the new branch is born with no upstream (reference:
                              # internal/gitpython_repository.py:59-87)
worktree_list(source)         worktree list --porcelain                    each line "worktree <path>": that path
common_dir(path)              rev-parse --path-format=absolute --git-common-dir    trimmed, resolved
```

```text
# crates/core/src/env_create/worktree.rs, pub (Domain, Visibility), also read by list.md: a worktree's tracking. Unlike
# tracking.md's head, a detached HEAD is no branch here: ws init refuses to wire it (reference:
# internal/gitpython_repository.py:140-188); tracking.md = ../../coordinated-git/spec/tracking.md
head_branch(path)             symbolic-ref --quiet --short HEAD            exit 1 -> None (detached)
setting(path, key)            config --get <key>                            exit 1 -> None
tracking(path)                head_branch; None -> None
                              remote := setting(branch.<head>.remote), merge := setting(branch.<head>.merge);
                              either None -> None; else "<remote>/<merge without a leading refs/heads/>"
                              # reference: internal/gitpython_repository.py:131-137 (GitPython tracking_branch)
set_upstream(path, u)         u split at its first "/" into remote and branch; an empty branch -> BadUpstream
                              head_branch None -> HeadDetached
                              config branch.<head>.remote <remote>; config branch.<head>.merge refs/heads/<branch>
                              # the config is written directly, so a never-pushed branch can be the upstream
                              # (internal/gitpython_repository.py:140-188)
```

### Running `ws init <env>`

```text
# reference: modules/workspace/handlers/init_handler.py:31-50, modules/workspace/init_service.py:223-306
order
  0. the command file: command-surface.md's ws init rules, a usage refusal before the call
  steps 1-4 are init_env's
  1. root discovery and the load; a failure is EnvInitError::Config, exit 1, nothing written
  2. env_name::check(TARGET) (env-names.md); a refusal is EnvInitError::EnvName, exit 1 with nothing read from git
     and nothing written
  3. the env index registry read (lock-and-state.md); a refusal is EnvInitError::Config, exit 1, nothing written
  4. reconcile_env(TARGET); Completed { success: false } -> exit 1, true -> 0; Stopped -> its ending
lock-and-state.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md
```

```text
# product:environment-worktrees; decision:parallel-workspaces/environment-lifecycle/allocation-policy
reconcile_env(name)
  TargetStarted name
  a. allocate         state_registry::allocate (allocation.md), reported as its Reporting row gives
                      an error -> RepoError, TargetCompleted false, and nothing below runs   # nothing is created
                      # registered deviation: the reference creates the directory first (init_service.py:255-266)
  b. directory        create_dir_all(<root>/<name>); an io::Error -> RepoError repo <name>, TargetCompleted false,
                      nothing below runs; the recorded index stays (no rollback, product:environment-recovery)
  c. self-exclude     write_block("winter-dir/<name>", ["/<name>/"]) (managed-content.md)
                      Some(file) -> RepoAction repo "winter", location file, WorkspaceExcludesUpdated, detail "/<name>/"
                      an error   -> RepoError repo "winter", "workspace exclude — <cause>"; the target fails, and the
                                    run goes on
  d. sources          each project, in merged order: <root>/projects/<p> absent -> RepoError SourceCheckoutMissing;
                      the target fails and that project is not attempted; the others are "ready"
  e. infer            over the ready projects that are not pinned and whose location exists: the set of tracking(l)
                      values that are Some; exactly one value -> inferred, else None
                      # runs before the fan-out, so a worktree created in this run never votes
                      # reference: init_service.py:665-690
  f. fan-out          worktree::run per ready project, as Fan-out below
  g. hooks            on_env_init of every extension, in invocation-contract.md's order, cwd <root>/<name>, run
                      whatever d or f reported; a failed hook fails the target
  TargetCompleted name, success: no RepoError was emitted for the target and every step succeeded
```

### One worktree

```text
# reference: modules/workspace/init_service.py:564-647,724-789
# each step after the one before succeeds, except as noted; the first failure is the project's RepoError event, and the
# event's text names the step that failed and the steps not attempted, in this order:
# location, identity, excludes, tracking, cmd, nested (product:environment-recovery)
worktree::run(w)
  1. location
     absent (nothing at w.location, links not followed):
       resolve(<root>/<env>) != resolve(root)/<env>, checked again immediately before worktree_add, so a directory
         replaced by a link since reconcile_env's step b is caught -> NotAWorktreeOf
       branch_exists(source, env) -> worktree_add(source, location, env)
                                  else worktree_add_new(source, location, env, main_branch)
       -> RepoAction WorktreeCreated            # a reused local branch and a new one report the same; step 4
                                                # knows which
     present:                                   # engineering:worktree-effect-confinement
       any of these -> NotAWorktreeOf
         resolve(location) != resolve(root)/<env>/<repo>     # a link to another environment's worktree included
         common_dir(location) != common_dir(source)
         toplevel(location) != resolve(location)
         resolve(location) is no path of worktree_list(source)
       -> RepoAction Exists
       # registered deviation: decision:parallel-workspaces/environment-lifecycle/location-confinement; the
       # reference reports exists without a check
  2. identity     reconciliation.md, One repository, step 2, at w.location
  3. excludes     reconciliation.md, One repository, step 3, at w.location; exclude_path follows the worktree's
                  .git file and commondir to the store's info/exclude
  4. tracking     its failure is the project's RepoError and fails the project, and step 5 still runs
                  # engineering:lifecycle-step-isolation; reference: init_service.py:600-628
     head:        head_branch(location) None -> RepoAction WorktreeDetached; the rest of step 4 runs as below, so
                    a write it needs fails as HeadDetached
                  Some(b), b != env -> RepoAction WorktreeParked, detail b; the rest of step 4 is skipped, so
                    another branch's tracking is never written
                  # decision:parallel-workspaces/environment-lifecycle/head-state: no step checks out <env>, so a
                  # detached or parked worktree is reported, never reattached (product:environment-worktrees)
     pinned:      desired := "origin/<main_branch>"; changes := []
                  tracking(location) != desired -> set_upstream(location, desired); changes += desired
                  push_default(location) != Some("upstream") -> set_push_default_upstream(location);
                    changes += "push.default=upstream"
                  changes not empty -> RepoAction PinnedTrackingSet, detail changes joined ", "
     not pinned:  inferred None -> nothing
                  current := tracking(location)
                  current == inferred -> nothing
                  current Some and step 1 did not create the branch (worktree_add_new) -> nothing
                    # operator-owned tracking, a reused local <env> branch's included; registered deviation:
                    # decision:parallel-workspaces/environment-lifecycle/reused-branch-tracking; the reference
                    # overrides the tracking of any worktree it adds (init_service.py:666-701)
                  else set_upstream(location, inferred); set_push_default_upstream(location);
                    RepoAction UpstreamInferred, detail inferred
                  # push-default.md, when each caller writes it: ws init
  5. cmd          reconciliation.md, Commands, cwd w.location
  6. nested       delegation.md, for a project with `nested` declared; skipped when step 5 failed
  -> true when no step reported a RepoError
```

### Fan-out

```text
# engineering:bounded-fan-out: at most four repository tasks at once; reference:
#   modules/workspace/init_service.py:308-344, internal/git_ops_service.py:24
workers     at most 4 projects at once; one project's steps in order on one worker; reconciliation.md, Fan-out, gives the
            channel, the sink, the stop on a sink error, and the threads
events      in arrival order across projects; one project's events keep their order
outcome     a project's failure never stops another project
```

### Running `ws init --all`

```text
# reference: modules/workspace/init_service.py:360-393, handlers/init_handler.py:41-46
order                                             # TARGET with --all is command-surface.md's refusal, step 0
  steps 1-5 are init_all's
  1. 1 and 3 of ws init <env>: root discovery, the load, and the registry read, each failure EnvInitError::Config
  2. the workspace part: bootstrap::reconcile (reconciliation.md) from the same start, its events and its exit
     contribution; its InitError is EnvInitError::Config or Internal, exit 1
  3. the registry findings of allocation.md, as warnings
  4. discover; a GitFailure -> EnvInitError::Repo or Internal, exit 1, after the events of 2
  5. reconcile_env for each discovered name, in order, every one attempted whatever an earlier one reported
  exit 1 when the workspace part failed or any reconcile_env completed without success; else 0; a Stopped run of
  either ends the invocation with its ending, and no later environment is attempted
```

### Discovery for `--all`

```text
# decision:parallel-workspaces/environment-lifecycle/discovery-authority
# reference: modules/workspace/init_service.py:367-393
discover()
  P := the first project entry in merged order; none -> []
  source := <root>/projects/<P>; absent -> []
  for each path of worktree_list(source):
    resolve(path) == resolve(source)                    -> skipped
    d := parent of resolve(path); parent of d resolved != resolve(root) -> skipped
    name := d's last component
    d is not a directory                                -> warning "worktree registration <path> names env <name>,
                                                           whose directory is gone: not recreated"; skipped
                                                           # registered deviation: the reference recreates the env
    env_name::check(name) fails                         -> warning naming the name and the fault; skipped
                                                           (env-names.md)
    otherwise                                           -> name
  the names sorted, without repeats
an environment that holds no worktree of P is not found, and a name found that the registry does not record is
recorded by its reconcile_env, as the reference does
```

## Flows

### Creating `alpha`, then reconciling it

```text
ws init alpha                                    ws init alpha (again)
→ alpha                                          → alpha
[winter] workspace excludes updated: /alpha/     [api] exists at <root>/alpha/api
[api] worktree created at <root>/alpha/api       [web] exists at <root>/alpha/web
[web] worktree created at <root>/alpha/web       [api] $ make setup                 # every cmd runs again
[api] $ make setup                               ✓ alpha done
✓ alpha done
```
