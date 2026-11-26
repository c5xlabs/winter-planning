# Teardown

The nested phases of `ws destroy`: reading every child an env holds, refusing on child work that would be lost,
previewing, retiring each child env through the child's own `ws destroy`, and stopping the child's workspace services,
all before any outer teardown effect. The command, its options, its target selection, its own safety check, and its
later phases (provision teardown, destroy hooks, worktree and directory removal, exclude and index release) are
`epic:parallel-workspaces/environment-lifecycle`'s ([its spec](../../environment-lifecycle/spec/destroy.md)); options
are the `ws destroy` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands.
Every child call is a call of [invocation.md](./invocation.md); `NestedState` is that of
[child-state.md](./child-state.md). Events are the shape C `InitEvent`s of
[reconciliation.md](../../workspace-bootstrap/spec/reconciliation.md), Events and text. Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/nested/teardown.rs — product:nested-teardown-preservation
// decision:parallel-workspaces/nested-workspaces/recursive-teardown
pub(crate) struct Child<'c> { pub project: &'c ProjectRepository, pub root: PathBuf, pub state: NestedState }
pub(crate) struct Forwarded { pub force: bool, pub strict: bool, pub provision_teardown: bool }
pub(crate) enum Audit<'c> { Read(Vec<Child<'c>>), Unreadable(Vec<String>) }   // Unreadable: project names
pub(crate) fn audit(nesting: &Nesting<'_>, worktrees: &[(&ProjectRepository, PathBuf)],
    sink: &mut dyn FnMut(InitEvent)) -> Audit<'_>;
pub(crate) fn losses(children: &[Child<'_>]) -> (Vec<String>, Vec<String>);          // (dirty, unpushed) entries
pub(crate) fn preview(nesting: &Nesting<'_>, children: &[Child<'_>], sink: &mut dyn FnMut(InitEvent)) -> bool;
pub(crate) fn retire_envs(nesting: &Nesting<'_>, children: &[Child<'_>], forwarded: Forwarded,
    sink: &mut dyn FnMut(InitEvent)) -> bool;
pub(crate) fn stop_services(nesting: &Nesting<'_>, children: &[Child<'_>], sink: &mut dyn FnMut(InitEvent)) -> bool;
```

```rust
// crates/core/src/bootstrap.rs — Action gains; location, detail, and text per action
WouldDestroyNestedEnv,            // "would_destroy_nested_env"   <root>/<env>, <env>
                                  //   "[<repo>] would destroy nested env <env> at <location>"
NestedEnvDestroyed,               // "nested_env_destroyed"       <root>/<env>, <env>
                                  //   "[<repo>] nested env <env> destroyed at <location>"
WouldStopNestedServices,          // "would_stop_nested_workspace_services"   <root>, ""
                                  //   "[<repo>] would stop nested workspace services at <location>"
NestedServicesStopped,            // "nested_workspace_services_stopped"      <root>, ""
                                  //   "[<repo>] nested workspace services stopped at <location>"
```

## Interfaces

### Phases

```text
# reference: modules/workspace/destroy_service.py:200-429,431-466, modules/workspace/nested_workspace_service.py:183-265
# events carry repo = the outer project's name, except the refusals and aborts, which carry the env's name
children   the env's worktrees that exist on disk, of projects with `nested` declared, in configuration order
order within one env target of ws destroy:
  0. audit      each child: nesting.state(root); a RepoError -> RepoError "nested workspace state — <error>", the
                project is unreadable, and the other children are still read
                any unreadable -> RepoError on the env "refusing to destroy — cannot read nested workspace: <names>;
                keeping the env.", whatever --force; the target fails with nothing torn down and no later phase
                runs                                                               # see Deviations
  1. losses     without --force, added to environment-lifecycle's own refusal lists, which refuse the target:
                dirty     "<repo> (nested: <names of the child's envs with a dirty repo, joined ', '>)"
                unpushed  "<repo> (nested: <unpushed() joined ', '>)"; the refusal says the work would be deleted
                          with the worktree and how to keep it
                # the audit is the one taken in phase 0, before any child teardown (engineering:nested-loss-evidence)
  --dry-run     after phase 1 and before environment-lifecycle's preview: preview, then stop
                each child, each env in its state's order: WouldDestroyNestedEnv
                each child: binds_service(root) -> WouldStopNestedServices when true; a RepoError -> RepoError
                "nested workspace services — <error>" and the preview fails
                no child CLI call other than the phase 0 status reads runs
  2. child envs each child in order, each env in its state's order, every one attempted:
                CmdStarted "winter ws destroy <env>"
                nesting.run(root, ["ws", "destroy", <env>] + ["--force" if force] + ["--strict" if strict]
                                  + ["--no-provision-teardown" if not provision_teardown])
                  RepoError -> RepoError "nested env <env> — <error>", no CmdCompleted
                  returncode != 0 -> CmdCompleted; RepoError "nested `winter ws destroy <env>` exited with code <n>"
                  0 -> CmdCompleted; NestedEnvDestroyed
                any failure -> RepoError on the env "aborting destroy — a nested env could not be destroyed;
                keeping the env.", whatever --force; the target fails, and phases 3 and 4 do not run   # see Deviations
                # the child's ws destroy runs the child's provision teardown and stops its per-env services
  3. services   each child in order: binds_service(root)
                  false -> nothing runs   # `service down` fails where no provider is bound
                  true  -> CmdStarted "winter service down workspace"; nesting.run(root, ["service", "down",
                           "workspace"]); RepoError -> RepoError "nested workspace services — <error>";
                           returncode != 0 -> CmdCompleted, RepoError "nested `winter service down workspace` exited
                           with code <n>"; 0 -> CmdCompleted, NestedServicesStopped
                  a RepoError of binds_service -> RepoError "nested workspace services — <error>"
                any failure -> RepoError on the env "aborting destroy — a nested workspace's services could not be
                stopped; keeping the env.", whatever --force; the target fails, and phase 4 does not run
                                                                                    # see Deviations
  4.            environment-lifecycle's provision teardown, hooks, and removals; none starts before phase 3 ends
children run serially: one child call at a time, as ../../coordinated-git/spec/fan-out.md gives destructive commands
```

```text
# product:nested-teardown-preservation; reference: modules/workspace/nested_workspace_service.py:227-237,326-333
binds_service(root)
  the child's .winter/config.toml and .winter/config.local.toml, each absent reads as empty, the local layered over
  the committed as configuration-model.md, Layering, gives
  either not valid TOML -> NestedConfigUnreadable
  true when capabilities is a table whose `service` is a non-empty string, or an array holding one
```

```text
# product:nested-teardown-preservation; decision:parallel-workspaces/nested-workspaces/recursive-teardown
Forwarded
  force               destroy.md's Options.force
  strict              destroy.md's Options.strict
  provision_teardown  destroy.md's Options.provision_teardown
  # --strict and --no-provision-teardown, and the two Options fields, are registered by the owners destroy.md,
  # System, names: epic:agent-ready-operation/executable-extensions and epic:running-environments/resource-provisioning;
  # each sets its Forwarded field from the Options field it adds
```

### Deviations

```text
# registered deviation: decision:parallel-workspaces/nested-workspaces/recursive-teardown, decision:rough-parity
# product:nested-teardown-preservation, engineering:nested-loss-evidence
phase 0   the reference refuses an unreadable child only without --force; with --force it skips that child, never
          destroys its envs, and removes the worktree holding them. Here --force does not bypass phase 0
phase 2   the reference goes on past a failed child env under --force and removes the worktree. Here a child env
          that could not be torn down keeps the outer env, whatever --force
phase 3   the reference goes on past a failed workspace-service stop under --force and removes the worktree the
          child's services may still run from. Here it keeps the outer env, whatever --force
the result names the cause in each: the RepoError events above and a failed target, exit 1
```
