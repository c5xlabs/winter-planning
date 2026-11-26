# Delegation

The nested step of `ws init <env>` and `ws init --all`: verifying the child in an env worktree of a project with
`nested` declared, writing the ports, prefix, and inherited settings the parent delegates into the child's local
overlay, and running the child's own bare `ws init`. The env worktree's other steps, the order of repositories, and the
fan-out are `epic:parallel-workspaces/environment-lifecycle`'s ([its spec](../../environment-lifecycle/spec/init.md));
the step runs after the worktree's `cmd`, whose scrubbed environment is `scrubbed` of [invocation.md](./invocation.md).
Shape C events are `InitEvent` of [reconciliation.md](../../workspace-bootstrap/spec/reconciliation.md), Domain. The
local-overlay edit is the "Upsert local keys" row of
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Writers, through the port of
[write-path.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/write-path.md); the env index is
that of [lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md).
Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message
wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/nested/delegate.rs — product:nested-runtime-allocation, product:nested-local-inheritance
// decision:parallel-workspaces/nested-workspaces/delegation-policy; reference: modules/workspace/nested_overlay.py
pub(crate) struct Slice { pub size: i64, pub base: i64 }
pub(crate) fn slice(ports_per_env: i64, nested_count: usize, position: usize, env_port_base: i64) -> Slice;
pub(crate) fn delegated(base: i64, prefix: &str, envs: Option<i64>) -> Vec<(&'static str, toml::Value)>;
pub(crate) fn inherited(outer_local: &toml::Table, names: &[String]) -> Vec<(String, toml::Value)>;
pub(crate) fn footprint(committed: &toml::Table, local_after: &toml::Table) -> i64;
pub(crate) fn reconcile(nesting: &Nesting<'_>, workspace: &Workspace, project: &ProjectRepository, root: &Path,
    env: &str, sink: &mut dyn FnMut(InitEvent)) -> Result<(), RepoError>;
```

```rust
// crates/core/src/bootstrap.rs — Action gains
NestedAllocationMoved,   // "nested_allocation_moved"  detail "base_port <old> -> <new>"; registered deviation below
NestedOverlayWritten,    // "nested_overlay_written"   detail below
// text: "[<repo>] nested allocation moves: <detail>", "[<repo>] nested config.local.toml updated: <detail>"

// crates/core/src/termination.rs — RepoMessage gains
DelegationRefused { repo, env, used, available, per_env, outer_ports_per_env, slices, raise_to, envs_fit: Option<i64> },
NestedCommandFailed { command, returncode },   // "nested `<command>` exited with code <returncode>"
```

## Interfaces

### Values

```text
# decision:parallel-workspaces/nested-workspaces/delegation-policy
# reference: modules/workspace/nested_overlay.py:17-83,
#   modules/workspace/nested_workspace_service.py:103-170, modules/workspace/env_index.py:155-168
K               the number of projects with `nested` declared, in the loaded configuration
position        1 + the project's index among them, in configuration order; slice 0 is the parent's own services
env_port_base   port_base(index of <env>) of configuration-model.md, Derived values, the index registry-first as
                lock-and-state.md, Env index allocation, records it for <env> by this point of `ws init`
slice           size = ports_per_env div (K + 1), floored; base = env_port_base + position * size; the remainder of
                the band is unused
delegated       base_port = base; service_prefix = "<service_prefix>-<env>-<project name>"
                when the project's `envs` is set: env_aliases = []; envs_per_workspace = envs + 1
                # with no aliases, index 1 is the buffer slot, so envs usable indices need envs + 1 slots
inherited       names = the project's inherit_local, or ["git"] when unset; for each name in order that is a key of
                the outer workspace's own .winter/config.local.toml as parsed, and is not project_repository or
                standalone_repository: that key's value, copied whole. The outer local file is read only when names
                is not empty; absent reads as empty. Neither committed keys nor defaults are copied
values          inherited, then delegated, a delegated key replacing an inherited one of the same name
footprint       (E + 1) * P, E and P the child's envs_per_workspace and ports_per_env, each from local_after, else the
                child's .winter/config.toml, when it is an integer and not a boolean, else 48 and 20
                local_after = the child's local overlay as parsed, with values set over it
```

### The nested step

```text
# product:verified-child-workspace; reference: modules/workspace/init_service.py:570-605,
#   modules/workspace/nested_workspace_service.py:88-140
per env worktree of a nested project, once its `cmd` succeeded; a `cmd` failure skips the step
  1. nesting.ensure_verified(root)                    any RepoError: nothing written, no child init
  2. read the outer local file (inherited), then the child's .winter/config.toml
                                                      not valid TOML -> NestedConfigUnreadable, nothing written
  3. WriteFiles::update(<root>/.winter/config.local.toml, edit), under the child's .winter/ lock; edit, on the bytes
     read under the lock (None reads as empty):
       not valid TOML                     -> NestedConfigUnreadable; no write
       footprint(committed, local_after) > slice.size -> DelegationRefused; no write
       the current local layer holds an integer base_port other than slice.base
                                          -> NestedAllocationMoved, detail "base_port <old> -> <slice.base>",
                                             sent before the write                    # see Moves
       the upsert of every key of values, configuration-model.md's row; no change in text -> no write
     replaced -> NestedOverlayWritten, detail "<key> = <value>" for each delegated key joined ", ", then
                 "; inherited <names joined ', '>" when inherited is not empty; values in TOML inline form
     an io::Error -> ConfigError::FileWrite of write-path.md, as this repository's RepoError
  4. CmdStarted "winter ws init"; nesting.run(root, ["ws", "init"]), each line a CmdOutputLine; CmdCompleted
     returncode != 0 -> NestedCommandFailed; the child creates no feature environment
any RepoError ends this repository as a RepoError event and fails its target; other repositories go on
every init rewrites values; a key the parent no longer delegates or inherits stays in the child as it is
```

```text
# DelegationRefused text, from the layers the check used
"nested workspace '<repo>' needs <used> ports for env '<env>' ((envs_per_workspace + 1) x ports_per_env of <P>), but
 its slice of the outer env band holds only <size> (outer ports_per_env of <ports_per_env> split into <K + 1> equal
 slices, the first for the parent's own services). Raise the outer `ports_per_env` to at least <used * (K + 1)>"
then ", or set `envs = <size div P - 2>` or fewer on its [[project_repository]] entry." when that number is >= 1,
else ", or lower the nested workspace's `ports_per_env`."
a slice of size 0 refuses every child whose footprint is above 0
```

### Moves

```text
# registered deviation: decision:parallel-workspaces/nested-workspaces/delegation-policy, decision:rough-parity
# the reference rewrites base_port without a word when adding, removing, or reordering nested projects moves a slice
# product:nested-runtime-allocation: the children that move are named before the write applies
moved      the child's local layer already holds base_port as an integer and slice.base differs from it
not moved  no base_port in the local layer (a first delegation), or the same value
the write still applies: the report names the move, it does not refuse it
```
