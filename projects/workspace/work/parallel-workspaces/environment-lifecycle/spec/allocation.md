# Allocation

Recording, moving, refusing, and releasing an environment's index in the env index registry, and the registry findings
the lifecycle commands report. It extends the frozen
[lock-and-state.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/lock-and-state.md): its file
format, read rules, layout, `suggested`, `lookup`, and probe stand as written there, and this leaf adds the allocation
write, the checks before it, and the release. Every write goes through the file port's `update` of
[write-path.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/write-path.md). `ws init <env>`
([init.md](./init.md)) allocates; `ws destroy` ([destroy.md](./destroy.md)) releases. Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/env_index.rs — gains, beside lookup; pure. product:stable-environment-allocation
// decision:parallel-workspaces/environment-lifecycle/allocation-policy
pub(crate) enum Allocation {
    Recorded { index: i64 },                         // the registry already records this index for the name
    New { index: i64 },                              // the name gets this index; the registry records it
    Moved { from: i64, to: i64 },                    // an alias recorded at another index takes its position
}
pub(crate) fn allocate(name: &str, aliases: &[String], envs_per_workspace: u32,
    registry: &std::collections::BTreeMap<String, i64>) -> Result<Allocation, EnvIndexError>;

pub(crate) enum RegistryFinding {                    // the warning text names every field
    OutOfRange { name: String, index: i64 },         // index <= 0 or index > M
    ReservedSlot { name: String, index: i64, slot: Reserved },   // a name that is no alias, recorded in 1 ..= N + 1
    AliasMoved { name: String, recorded: i64, position: i64 },   // an alias recorded off its position
    Shared { index: i64, names: Vec<String> },       // two or more names recorded at one index, in file order
}
pub(crate) enum Reserved { Alias { alias: String }, Buffer }
pub(crate) fn findings(aliases: &[String], envs_per_workspace: u32, registry: &[(String, i64)]) -> Vec<RegistryFinding>;

// crates/core/src/state_registry.rs — gains; each is one update of write-path.md on <root>/.winter/state.toml
// engineering:allocation-integrity: the read, the decision, and the write happen under the one .winter/ lock
pub(crate) fn allocate(files: &dyn WriteFiles, root: &Path, name: &str, aliases: &[String],
    envs_per_workspace: u32) -> Result<Allocation, AllocateError>;
pub(crate) fn release(files: &dyn WriteFiles, root: &Path, name: &str) -> Result<Option<i64>, ConfigError>;
                                                     // Some(index): the name was recorded and is removed
pub(crate) enum AllocateError { Index(EnvIndexError), State(ConfigError) }
```

```rust
// crates/core/src/env_index.rs — EnvIndexError (lock-and-state.md, Domain) gains; registered deviations below
#[error("env {name} is recorded at index {index}, which is {slot}: refusing to share it")]
RecordedReserved { name: String, index: i64, slot: Reserved },   // "the slot of alias <alias>" | "the buffer slot"
#[error("alias {name} takes index {index}, which the registry records for {holder}")]
AliasSlotHeld { name: String, index: i64, holder: String },
```

## Interfaces

### Allocate

```text
# reference: modules/workspace/env_index.py:180-243; lock-and-state.md, Env index allocation, holds the layout
# decision:parallel-workspaces/environment-lifecycle/allocation-policy; registered deviations marked
N = len(aliases)    M = envs_per_workspace    position(a) = lock-and-state.md's alias index (the later of repeats)
allocate(name)
  name is an alias, p = position(name)
    another name recorded at p                        -> AliasSlotHeld { holder: the first such name in file order }
                                                         # registered deviation: the reference records p beside it
    name recorded at r, r != p                        -> Moved { from: r, to: p }   # registered deviation: the
                                                         reference moves it without a word
    name recorded at p                                -> Recorded { index: p }
    otherwise                                         -> New { index: p }
  name recorded at r
    1 <= r <= N                                       -> RecordedReserved { slot: Alias { the alias at r } }
    r == N + 1                                        -> RecordedReserved { slot: Buffer }
                                                         # registered deviation: the reference answers r
    otherwise                                         -> Recorded { index: r }      # r <= 0 or r > M included;
                                                         findings report it
  otherwise                                           -> the probe of lock-and-state.md over the registry's indexes:
                                                         New { index }, or BandFull
the probe and the checks read the registry as the update reads it under the lock, never an earlier read
```

```text
state_registry::allocate(files, root, name, ...)
  update(<root>/.winter/state.toml, edit):
    edit(current)
      parse current as lock-and-state.md, .winter/state.toml reads it; a refusal -> AllocateError::State
        (ConfigError::FileRead naming the file), nothing written
      env_index::allocate(...) Err -> AllocateError::Index, nothing written
      Recorded -> no write (edit returns None)
      New or Moved -> the parsed document with env_index[name] = index, serialised with the toml crate
        (engineering:toml-library): every other top-level key and every other entry kept, a missing or non-table
        env_index replaced by a table, its other entries in their order and the new name last; an inline env_index
        is written back as a standard table holding its entries and the new name
        # registered deviation: decision:parallel-workspaces/workspace-configuration/inline-registry-table; the
        # reference drops an inline table's entries
        # meaning, not bytes, as decision:toml-parity: comments and layout of the old file are not kept
  an io::Error of update -> AllocateError::State(ConfigError::FileWrite naming the file)
  # two invocations allocating at once serialise on the lock; the second sees the first's record, so it probes past it
```

### Release

```text
# reference: modules/workspace/internal/toml_env_index_registry.py:58-64
release(files, root, name)
  update(state file, edit): parse as above; name not recorded -> no write, Ok(None); else the document without that
  entry -> Ok(Some(index)); an emptied env_index stays an empty [env_index] table
  # lock-and-state.md, Open questions, asks what the reference leaves; the binary's empty table reads the same
called only once the env's worktrees are removed: destroy.md, Per target   # teardown-policy
```

### Findings

```text
# product:environment-state-recovery: each is reported by name
findings(aliases, M, registry), in this order, each kind in registry file order:
  OutOfRange     r <= 0 or r > M
  ReservedSlot   name is no alias and 1 <= r <= N + 1
  AliasMoved     name is an alias and r != position(name)
  Shared         an index recorded for two or more names; once per index, the names in file order
written as one warning record each, origin "env_index", level warning, through Diagnostics; never an event, never a
change of exit code
written by: ws list (list.md), before its result; ws init --all (init.md), once, before its first environment
```

### Reporting an allocation in `ws init <env>`

```text
Recorded, New          no event
Moved { from, to }     before the write is applied: RepoAction { repo: <env>, location: <root>/.winter/state.toml,
                       action: IndexMoved, detail: "<from> -> <to>" }; then the write; the run goes on
                       # registered deviation: the reference moves the env's ports silently
EnvIndexError          the target's RepoError event (its rendered error), the target fails, and nothing of the target
                       is created: the allocation runs before the env directory exists (init.md)
                       # registered deviation for BandFull: the reference creates the directory first and leaves it
AllocateError::State   the same: RepoError naming the file, the target fails, nothing created
```

```rust
// crates/core/src/bootstrap.rs — Action gains
IndexMoved,   // "index_moved"   text "[<env>] index moved: <detail>"
```
