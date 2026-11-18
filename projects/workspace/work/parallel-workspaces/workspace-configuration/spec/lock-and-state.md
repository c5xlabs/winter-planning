# Lock and state files

The two files the binary owns outright: the standalone pin lock and the machine-local env index registry. Both keep the
reference tool's file format (`decision:configuration-transition`). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`. The declared configuration is
[configuration-model.md](./configuration-model.md), which also owns write integrity for these files.

Each part is built with the first command that uses it: the lock file, read and written, by
`epic:parallel-workspaces/reproducible-inputs`; the registry's write, allocation, and release by
`epic:parallel-workspaces/environment-lifecycle`. Reading the registry and the lookup are built, with `ws index`.

The file contents are contract, because the lock is committed and both files are read back by either tool. The wording
of the errors below is the binary's own (`decision:functional-parity`).

## Domain

```rust
// engineering:typed-outcomes
#[derive(Debug, thiserror::Error)]
pub enum LockError {                   // carried as ConfigError::Lock; reference: config_lock_repository.py:20
    #[error("cannot load {path}: {cause}")]
    Load { path: PathBuf, #[source] cause: FileCause },           // unreadable, or not valid TOML
    #[error("{path}: unsupported lock file version")]
    UnsupportedVersion { path: PathBuf, found: Option<ConfigValue> },
    #[error("{path}: malformed [[standalone]] entry")]
    MalformedEntry { path: PathBuf, fault: LockEntryFault },
}
pub enum LockEntryFault {
    MissingField { field: &'static str },      // name, ref, kind, commit
    InvalidKind { value: String },
}

#[derive(Debug, thiserror::Error)]
pub enum EnvIndexError {               // reference: modules/workspace/env_index.py:240-243
    #[error("cannot allocate an index for env {name}: all {band_size} slots of the hash band {first}..{last} are occupied")]
    BandFull { name: String, band_size: u32, first: u32, last: u32 },
}
```

`FileCause` and `ConfigValue` are defined in [configuration-model.md](./configuration-model.md). `path` is the absolute
path of the lock file. The `#[error]` texts above show which values a message carries; they are not fixed.

## Data

### `.winter/config.lock`

```toml
# .winter/config.lock — managed by winter; commit this file.
version = 1

[[standalone]]
name = "winter-context"
ref = "main"
kind = "branch"
commit = "0123456789abcdef0123456789abcdef01234567"
```

| Element          | Rule                                                                                                                                                                                                                               |
| ---------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Header           | The comment line above, byte for byte, is the first line of every written file, so a lock rewritten by either tool does not change. It keeps the literal `winter` (`decision:parallel-workspaces/component-runtime/program-name`). |
| `version`        | The integer `1`.                                                                                                                                                                                                                   |
| `[[standalone]]` | One block per pinned standalone, sorted by `name`, each preceded by one blank line.                                                                                                                                                |
| Field order      | `name`, `ref`, `kind`, `commit`.                                                                                                                                                                                                   |
| `kind`           | `branch`, `tag`, or `commit`.                                                                                                                                                                                                      |
| `commit`         | The 40-character object id the `ref` resolved to.                                                                                                                                                                                  |
| Empty lock       | The header line and `version = 1`, then end of file; no `[[standalone]]` array.                                                                                                                                                    |

| Read case                                                             | Result                          | Reference                         |
| --------------------------------------------------------------------- | ------------------------------- | --------------------------------- |
| File missing                                                          | An empty lock.                  |                                   |
| File exists and cannot be read                                        | `LockError::Load`, cause `Io`   | `config_lock_repository.py:54-58` |
| Not valid TOML                                                        | `LockError::Load`, cause `Toml` | `config_lock_repository.py:54-58` |
| `version` absent or not `1`                                           | `LockError::UnsupportedVersion` | `config_lock_repository.py:63`    |
| A `[[standalone]]` block missing a field or holding an unknown `kind` | `LockError::MalformedEntry`     | `config_lock_repository.py:74`    |

Each is a configuration error, exit 1. Every write is a full rewrite from the merged entry set: read, upsert by `name`,
sort, write (`modules/workspace/internal/config_lock_repository.py:14-109`). Writers: `ws init` (`init_service.py:554`),
`ws pull` and `ws update` (`workspace_sync_service.py:677,787`).

### `.winter/state.toml`

```toml
[env_index]
alpha = 1
feature-x = 14
```

| Case                                     | Behaviour                                                                                                                                                   |
| ---------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| File missing                             | An empty registry.                                                                                                                                          |
| TOML parse failure                       | **An empty registry, with no message** (`decision:parallel-workspaces/workspace-configuration/state-file-compatibility`). The next write replaces the file. |
| `env_index` value that is not an integer | That entry is ignored.                                                                                                                                      |
| `env_index` that is not a table          | Read as empty; replaced by a fresh table on the next write.                                                                                                 |
| Removing a name that is absent           | No write.                                                                                                                                                   |
| Other top-level keys                     | Preserved across a write.                                                                                                                                   |
| Parent directory missing                 | Created by the write.                                                                                                                                       |

Each write is load, modify, store (`modules/workspace/internal/toml_env_index_registry.py:51-89`). The file is
machine-local and never committed.

## Flows

### Env index allocation

```text
# product:parallel-workspaces/environment-lifecycle/stable-environment-allocation
# reference: modules/workspace/env_index.py:71-243
N    = len(env_aliases)                    M = envs_per_workspace
0                the workspace scope
1 ..= N          the aliases, by position in env_aliases
N + 1            buffer; never assigned
N + 2 ..= M      hash band; band_size = M - N - 1

suggested(name) = (N + 2) + ( u16_big_endian( sha1(utf8(name))[0..2] ) mod band_size )

allocate(name):
    name is an alias             -> its fixed index; recorded in the registry
    name is in the registry      -> its recorded index
    otherwise                    -> start = suggested(name); candidate = start
                                    while candidate is taken:
                                        candidate = (N + 2) + ((candidate - (N + 2) + 1) mod band_size)
                                        candidate == start -> EnvIndexError::BandFull
                                    record candidate
lookup(name): registry first; absent -> alias position, else suggested(name), with nothing recorded
release(name): remove from the registry
```

Configuration validation guarantees `M >= N + 2`, so the band always holds at least one slot and an empty band cannot be
reached.

### Band exhaustion

Allocation runs only while `ws init` reconciles a feature env, right after the env's root directory is created
(`modules/workspace/init_service.py:258`), so a full band is a failure of that target, reported in-band and not as a
command error (`init_service.py:237-245`).

```text
stderr:   a diagnostic naming the env and the full band
event:    repo_error for <name>; its `error` value is message text
          target_completed for <name> with success = false
target:   failed; nothing is recorded in the registry; the remaining targets still run; the run exits 1
```

The in-band internal-failure rule is in
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md).

### Fixed points

Under the default configuration (ten aliases, `envs_per_workspace = 48`, band `12..=48`, 37 slots):

| Name         | Index |
| ------------ | ----- |
| `alpha`      | 1     |
| `kappa`      | 10    |
| `my-feature` | 27    |
| `lambda`     | 28    |
| `feature-x`  | 40    |
| `a`          | 42    |

The index feeds `port_base(index)` in [configuration-model.md](./configuration-model.md) and `WINTER_ENV_INDEX` in the
[invocation contract](../../../agent-ready-operation/executable-extensions/spec/invocation-contract.md).

## Open questions

- What `state.toml` holds after its last name is removed — a bare `[env_index]` header or an empty file — is not
  confirmed from source. Engineering records it with the first `ws destroy` scenario.
