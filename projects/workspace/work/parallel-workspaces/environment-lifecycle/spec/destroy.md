# Environment teardown

The `ws destroy` command: selecting environments, refusing those whose work would be lost, previewing, and tearing each
down in a fixed order: nested children, provision teardown, destroy hooks, worktree removal, directory removal, the
self-exclude block, and the index. Its options are the `ws destroy` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands, and
its pattern shape that file's env-level shape; shape C and the no-match line are those of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md); the prompt is
the confirmation of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md),
Confirmation, under the thresholds of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Interrupts and prompts. Events are the `InitEvent`s of
[reconciliation.md](../../workspace-bootstrap/spec/reconciliation.md), Events and text. Names pass
[env-names.md](./env-names.md); selection is the Named rule of
[target-patterns.md](../../coordinated-git/spec/target-patterns.md); the order of effects is
[fan-out.md](../../coordinated-git/spec/fan-out.md)'s for destructive commands; the release is
[allocation.md](./allocation.md)'s. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
`winter-workspace` at `127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## System

```text
commands/ws/destroy.rs ── select ── confirm ──► env_destroy::destroy(ports, env, diagnostics, workspace, name, opts, sink)
                                                   for each selected env, serially, every one attempted
```

Phases that other epics own, at the position the per-target order below gives them:

| Phase                                                    | Position   | Owner                                              |
| -------------------------------------------------------- | ---------- | -------------------------------------------------- |
| Child audit, child losses, child preview, child teardown | 3, 4, 5, 6 | `epic:parallel-workspaces/nested-workspaces`       |
| Provision teardown run and its dry-run plan              | 5, 7       | `epic:running-environments/resource-provisioning`  |
| `on_env_destroy` hooks, run as invocation-contract.md    | 8          | `epic:agent-ready-operation/executable-extensions` |

The nested phases are [teardown.md](../../nested-workspaces/spec/teardown.md); invocation-contract.md is
[../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md](../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md).

## Domain

```rust
// crates/core/src/env_destroy.rs — product:destroy-environments, product:environment-recovery
// decision:parallel-workspaces/environment-lifecycle/teardown-policy
pub(crate) struct Options { pub force: bool, pub strict: bool, pub dry_run: bool, pub provision_teardown: bool }
pub(crate) fn select(ports: &Ports<'_>, workspace: &Workspace, registry: &[(String, i64)], patterns: &[String])
    -> Result<Vec<String>, CommandError>;
pub(crate) fn destroy(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, workspace: &Workspace,
    name: &str, options: Options, sink: &mut InitSink<'_>) -> Result<bool, Termination>;
    // Ok(success of the target); Err only for a sink that cannot write

// what the env directory holds, read once before any effect; engineering:teardown-evidence-completeness
pub(crate) struct Inventory {
    pub declared: Vec<(ProjectRepository, PathBuf)>,   // projects whose <root>/<env>/<name> is a directory, merged order
}

// crates/core/src/files.rs — WriteFiles gains; OsFiles implements it with std::fs::remove_dir
fn remove_empty_dir(&self, path: &Path) -> io::Result<()>;   // fails, removing nothing, unless the directory is empty
```

```rust
// crates/core/src/bootstrap.rs — Action gains; location, detail, and text
WorktreeRemoved,              // "worktree_removed"   <root>/<env>/<repo>; "" or "no source checkout"
                              //   "[<repo>] worktree removed at <location>" + " (<detail>)" when detail is not empty
EnvRemoved,                   // "env_removed"        <root>/<env>; ""      "[<env>] env directory removed at <location>"
WouldRemoveWorktree,          // "would_remove_worktree"  <root>/<env>/<repo>; ""
                              //   "[<repo>] would remove worktree at <location>"
WouldRemoveEnv,               // "would_remove_env"   <root>/<env>; ""      "[<env>] would remove env directory at <location>"
WouldRemoveWorkspaceExclude,  // "would_remove_workspace_exclude"  the exclude file; "/<env>/"
                              //   "[<env>] would strip workspace exclude block: <detail>"
EnvMissing,                   // "env_missing"        <root>/<env>; "index <i> released" | "index <i> would be released"
                              //   "[<env>] env directory already gone: <detail>"   # registered deviation, below
EnvKept,                      // "env_kept"           <root>/<env>; the leftover entry names, sorted, joined ", "
                              //   stderr "[<env>] warning: kept <location>, which still holds: <detail>"
                              //   registered deviation, below
// the workspace_excludes_updated action of reconciliation.md carries the strip: repo <env>, detail "removed /<env>/"
// provision teardown actions: Provision teardown below

// crates/core/src/termination.rs — RepoMessage gains
EnvDirectoryMissing { name, path },                  // "env directory not found at <path>"
Dirty { env, entries: Vec<String> },                 // "refusing to destroy — dirty worktrees: <entries>. Re-run with
                                                     //  --force to bypass."
InventoryUnreadable { env, path, cause },            // the env directory could not be listed
StrictAbort { env, phase: StrictPhase },             // "aborting destroy — <phase> failed and --strict was set"
WorktreesKept { env, failed: Vec<String> },          // "keeping <root>/<env> and its index: <failed> could not be
                                                     //  removed"                                    # registered deviation
ProvisionTeardown { env, stage: String, cause: ProvisionCause },
pub enum StrictPhase { ProvisionTeardown, Hook }     // "provision teardown", "on_env_destroy hook"
pub enum ProvisionCause { Config(String), Status(String) }   // "config error: <text>", "failed with status '<status>'"
// GitOperation gains WorktreeRemove
```

## Interfaces

### Selection

```text
# decision:parallel-workspaces/target-selection, product:target-patterns
# reference: modules/workspace/handlers/destroy_handler.py:51-100, modules/workspace/pattern_match.py:48-68
order, before any prompt, git call, or write
  1. root discovery and the load; a failure is its ConfigError, exit 1
  2. the env-level pattern shape (command-surface.md); a refusal is exit 1
  3. env_name::check(each literal), argument order (env-names.md)
  4. the registry read (lock-and-state.md); a refusal is its ConfigError, exit 1
  5. select:
     the Named rule of target-patterns.md, Reaching environments, over the env-level PATTERNS as env halves:
     literal   a PATTERN with none of * ? [; known_env (target-patterns.md) must hold, in pattern order: the first
               that does not -> SelectionError::UnknownEnv, exit 1, nothing done
               # decision:parallel-workspaces/environment-lifecycle/discovery-authority: a directory or a record
               # registered deviation: the reference prompts, then fails that target with "env directory not found"
     glob      the environments present (location-listing.md) whose name matches; env_name::check of each match
               (env-names.md)
     names     the literals sorted by code point, then the glob matches not among them, sorted, without repeats
  6. no names (only globs were given, and none matched): the line "No environments matched: <patterns joined by ' '>"
     on stdout, also under --json; exit 0 (output-formats.md, JSON-mode departures)
  7. prompt when more than one PATTERN was given or any PATTERN is a glob, unless --force or --dry-run:
     "This will destroy <n> environment(s): <names joined ', '>", then confirm("Continue?"); declined or end of input
     -> Termination::Aborted, exit 1, nothing done          # decision:parallel-workspaces/destructive-approval
  8. destroy(name) for each name, in order, one at a time; every one attempted whatever an earlier one reported
     # decision:parallel-workspaces/partial-execution: attempt all, exit 1; target_started and target_completed
     # name each completed and failed target, and a stop on an unwritable stdout leaves the rest unattempted
  exit 1 when any destroy returned false; else 0
lock-and-state.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md
location-listing.md = ../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md
```

### Per target

```text
# reference: modules/workspace/destroy_service.py:192-429
# one --force approves every bypass below and passes git's own --force (decision:parallel-workspaces/destructive-approval)
destroy(name)
  TargetStarted name
  1. directory    d := <root>/<name>
                  d is not a directory:
                    the registry records i for name:
                      --dry-run -> RepoAction EnvMissing "index <i> would be released"; TargetCompleted true
                      else      -> the strip of step 12, then release (allocation.md); RepoAction EnvMissing
                                   "index <i> released"; TargetCompleted true, or false when the strip or release
                                   failed            # registered deviation: the reference fails, exit 1, and the
                                                     # slot can never be released (destroy_service.py:203-208)
                    not recorded -> RepoError EnvDirectoryMissing; TargetCompleted false   # gone since selection
                  resolve(d) != resolve(root)/<name> -> RepoError UnsafeLocation (reconciliation.md) "outside the
                    workspace"; TargetCompleted false; nothing below runs   # engineering:worktree-effect-confinement
                    # registered deviation: decision:parallel-workspaces/environment-lifecycle/location-confinement; the
                    # reference removes the worktrees, then fails at its rmtree of the link
  2. inventory    entries(d); an io::Error -> RepoError InventoryUnreadable; TargetCompleted false
                  declared   each project, merged order, whose <d>/<name> is a directory, links followed
                             a project with nothing there: a warning naming <d>/<name>, and the project is skipped
                             (product:target-patterns)
  3. nested audit                       teardown.md, phase 0
  4. dirty        unless --force: entries := the name of each declared (repo, p) with git::clean(p) false, then
                  the child entries of teardown.md, phase 1
                  # git::clean reads every git failure as dirty, so a failed check never reads as nothing at risk
                  # (engineering:teardown-evidence-completeness)
                  any dirty entry -> RepoError Dirty; any unpushed child entry -> teardown.md's refusal
                  either -> TargetCompleted false; nothing below runs
  5. --dry-run    child preview (teardown.md); then the provision plan (Provision teardown, unless
                  --no-provision-teardown); then WouldRemoveWorktree per declared; WouldRemoveEnv d (removed at step 11 only when empty); then
                  locate (managed-content.md): a GitFailure -> RepoError "workspace exclude — <cause>", the preview
                  fails; a file whose lines hold Markers::exclude("winter-dir/<name>").begin ->
                  WouldRemoveWorkspaceExclude
                  TargetCompleted, true unless a preview step failed; stop
                  # the preview runs no teardown command and writes nothing; hooks, leftovers, and the index
                  # are not listed (product:destroy-environments)
  6. children     teardown.md, phases 2 and 3; a failure keeps the env as teardown.md gives
  7. provision    unless --no-provision-teardown: Provision teardown below
                  a failure with --strict -> RepoError StrictAbort(ProvisionTeardown); TargetCompleted false; stop
                  a failure without --strict -> remembered; the target fails at the end; teardown goes on
  8. hooks        on_env_destroy of every extension, invocation-contract.md's order, cwd d
                  a failure with --strict -> RepoError StrictAbort(Hook); TargetCompleted false; stop
                  a failure without --strict -> remembered; the target fails at the end; teardown goes on
                  # registered deviation: the reference exits 0 on a non-strict hook failure (destroy_service.py:384-401)
  9. worktrees    each declared (repo, p), in merged order, one at a time:
                    <root>/projects/<repo> absent -> WriteFiles::remove(p); RepoAction WorktreeRemoved "no source
                                                     checkout"; an io::Error -> RepoError, failed
                    else worktree_remove(source, p, --force) ; a GitFailure -> RepoError, failed
                         -> RepoAction WorktreeRemoved ""
 10. kept         any worktree failed -> RepoError WorktreesKept naming them; TargetCompleted false; stop: d, the
                  self-exclude block, and the index stay      # teardown-policy; registered deviation: the reference
                                                              # removes d and releases the index (destroy_service.py:403-421)
 11. directory    entries(d) after the removals; an io::Error -> RepoError "removing env directory — <cause>", the
                  target fails, and steps 12 and 13 still run
                  empty     -> remove_empty_dir(d) -> RepoAction EnvRemoved; an io::Error, an entry having appeared
                               since included -> RepoError "removing env directory — <cause>", the target fails, and
                               steps 12 and 13 still run
                  not empty -> RepoAction EnvKept naming every entry; d and all it holds stay, whatever --force; the
                               target does not fail for it
                  # decision:parallel-workspaces/environment-lifecycle/teardown-policy: Winter never deletes what it did
                  # not create; registered deviation: the reference removes d with all it holds (destroy_service.py:523-532)
 12. self-exclude locate; NotARepository or no file -> nothing; a GitFailure -> RepoError "workspace exclude — <cause>"
                  update(file): strip(content, Markers::exclude("winter-dir/<name>")); unchanged -> no write, no event
                  changed -> RepoAction repo name, location file, WorkspaceExcludesUpdated, "removed /<name>/"
                  an io::Error -> RepoError "<file> — <cause>"
                  a failure fails the target; step 13 still runs
 13. index        release(name) (allocation.md); a ConfigError -> RepoError naming the file, the target fails
                  # the worktrees are gone, so the environment no longer exists and its slot is reusable, also when
                  # step 11 kept leftovers: decision:parallel-workspaces/environment-lifecycle/teardown-policy
  TargetCompleted name, success: no RepoError was emitted for the target and nothing was remembered as failed
git
  worktree_remove(source, p, force)   worktree remove [--force] <p>     cwd source; --force exactly when --force was given
interrupt
  SIGINT ends the invocation as runtime.md, Interrupts and signals, gives; a re-run finds what remains: removed
  worktrees are no longer declared on disk, and a directory already gone releases its index at step 1
  (product:environment-recovery)
```

### Provision teardown

```text
# reference: modules/workspace/destroy_service.py:32-139,431-466; the run itself is resource-provisioning's
stages     "data", then "resource", each the provision run with action destroy, no seed, no service check
events     the first stage's start -> RepoAction repo <env>, location <env>, "provision_teardown_started",
                                      detail "data → resource"
           a stage's start         -> RepoAction repo <env>, location <stage>, "provision_subtarget_started", ""
           a stage with no handler -> RepoAction repo <env>, location <stage>, "provision_no_handlers", ""
           a handler's result      -> RepoAction repo <env>, location "<source>/<stage>[<scope>]",
                                      "provision_handler_done", detail the handler's action
           a handler's warning     -> RepoAction, same location, "provision_handler_warn", detail the warning
           a command               -> CmdStarted, CmdOutputLine, CmdCompleted, label and command as the run gives;
                                      an execution error -> RepoError with that label
           --dry-run, per handler  -> RepoAction, same location, "would_provision_teardown",
                                      detail "<action>: <commands joined ' && '>"
           after both stages       -> RepoAction repo <env>, location <env>, "provision_teardown_finished",
                                      detail "ok" or "error"; under --dry-run always "ok"
failure    a stage whose configuration cannot be loaded -> RepoError ProvisionTeardown Config; the next stage still runs
           a stage whose status is not "ok" -> RepoError ProvisionTeardown Status
text       every provision_* and would_provision_teardown action: "[<repo>] <action> <detail>", trimmed, as
           reconciliation.md's text of an action it does not list
```

## Data

```text
removed by a completed target              each declared worktree; <root>/<env> when nothing else is left in it;
                                           the "winter-dir/<env>" exclude block; the env's entry in .winter/state.toml
kept, and listed as env_kept               every other entry of <root>/<env>: files, clones, worktrees, residue
kept                                       each project's <env> branch in its source checkout (git worktree remove
                                           keeps branches); the stash list and history of the shared store
not changed                                sibling environments, projects/, standalones, workspace-scope services
```

## Open questions

- The provision run's events and failure results are resource-provisioning's contract, and that epic has no spec yet;
  the mapping above is the reference's adapter. Resource-provisioning engineering to confirm the run's events when its
  spec is written.
