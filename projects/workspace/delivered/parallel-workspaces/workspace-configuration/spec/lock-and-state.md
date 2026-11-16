# Lock and state files

The two files the binary owns outright: the standalone pin lock and the machine-local env index registry. Both keep the
reference tool's file format (`decision:configuration-transition`). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`. The declared configuration is
[configuration-model.md](./configuration-model.md); the replacement of either file is [write-path.md](./write-path.md).

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
sort, write (`modules/workspace/internal/config_lock_repository.py:14-109`). Writers: `ws init`, for each standalone
that has a `ref` (`init_service.py:550-554`); `ws pull`, when a branch pin fast-forwards
(`workspace_sync_service.py:493-496,770-787`); `ws update` (`workspace_sync_service.py:746`); and `ws update --freeze`,
which first writes the resolved commit as the entry's `ref` into the configuration file that declares it
(`workspace_sync_service.py:670-678`).

On a read, `version` is accepted when it compares equal to one as the reference tool compares it: the integer `1`, the
float `1.0`, and `true`. A command that tolerates a lock it cannot read does so by its own rule, in the epic that owns
the command. No command of this epic reads or writes the lock
(`decision:parallel-workspaces/workspace-configuration/unbuilt-readers-and-writers`).

### `.winter/state.toml`

```toml
[env_index]
alpha = 1
feature-x = 14
```

| Case                                     | Behaviour                                                                                                                                                  |
| ---------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------- |
| File missing                             | An empty registry.                                                                                                                                         |
| A text the parser refuses                | A configuration error which names the file, exit 1, whatever the parser's reason for refusing the text; the reference tool reads some such texts as empty. |
| `env_index` value that is not an integer | That entry is ignored, except a boolean, which is read as the block below states.                                                                          |
| `env_index` that is not a table          | Read as empty; replaced by a fresh table on the next write.                                                                                                |
| Removing a name that is absent           | No write.                                                                                                                                                  |
| Other top-level keys                     | Preserved across a write.                                                                                                                                  |
| Parent directory missing                 | Created by the write.                                                                                                                                      |

Each write is load, modify, store (`modules/workspace/internal/toml_env_index_registry.py:51-89`). The file is
machine-local and never committed.

```text
# decision:parallel-workspaces/workspace-configuration/state-read-failures
# decision:parallel-workspaces/workspace-configuration/state-file-compatibility
# reference: modules/workspace/internal/toml_env_index_registry.py:70-85
a path at which nothing exists              -> an empty registry
a path that exists and cannot be read,
  a directory at the path included          -> ConfigError::FileRead { path, cause: Io }                          exit 1
a file that is read and is not UTF-8        -> ConfigError::FileRead { path, cause: Io }, the cause being of the
                                               kind "invalid data" and carrying the decoding error               exit 1
a UTF-8 text that does not parse as TOML    -> ConfigError::FileRead { path, cause: Toml }                        exit 1
a UTF-8 text that parses                    -> the registry its env_index holds                  # table above
```

One test decides: the text parses or it does not. The binary reads nothing of its parser's error to decide, neither a
message nor a position; the error is carried as the cause. A text the two tools' parsers read differently is a text of
`decision:parallel-workspaces/workspace-configuration/toml-dialect`.

The reference tool draws the line elsewhere, and where it reads as empty a text the binary refuses, the difference is
the registered deviation of `decision:parallel-workspaces/workspace-configuration/state-read-failures`. Its registry
reads a parse error as an empty registry and catches nothing else (`toml_env_index_registry.py:82-85`). Its parser,
tomlkit 0.14.0, raises a parse error for a syntax error, such as a table header left open (`parser.py:913-916`). It
turns an error of another kind into a parse error only where it adds an item to the document itself, which is where a
top-level pair and a table section are added (`parser.py:146-149,160-163`); a key defined twice there
(`container.py:203-205,265,276`) is read as an empty registry. A key defined twice below it, while the body of a table
section is parsed, raises an error of another kind (`exceptions.py:191`) that the registry does not catch, so the
command ends unhandled, exit 1, as the binary's refusal does.

```text
examples of texts that do not parse, with what the reference tool does; the binary refuses every one, exit 1
    a syntax error, such as `[env_index` left open                        an empty registry    # registered deviation
    a key/value pair repeated above the first table header                an empty registry    # registered deviation
    a table header that repeats an earlier one                            an empty registry    # registered deviation
    a top-level table header over a top-level key that holds a value      an empty registry    # registered deviation
    a key/value pair repeated in the body of a table section              unhandled, exit 1
        parser.py:1005, items.py:1666, container.py:276
    a key repeated inside an inline table                                 unhandled, exit 1
        parser.py:654-655
    a child table header over a key of its section that holds a value     unhandled, exit 1
        parser.py:1010-1013, container.py:203-205
    a dotted key over a key of its section that holds a value             unhandled, exit 1
        parser.py:1005, container.py:170-171,133,203-205
    a dotted key defined twice in the body of a table section             unhandled, exit 1
        items.py:1668-1670, container.py:476,853,319
the list decides nothing: the rule is the one line of the block above, and a text that is not listed follows it
```

A file that is not UTF-8 carries the `Io` cause and not the `Toml` cause a configuration file carries for the same
bytes; the `Toml` cause of this file is carried by every text that does not parse.

```text
# reference: modules/workspace/internal/toml_env_index_registry.py:42-67
env_index written as an inline table        -> read as a registry
        what an allocation then writes      -> open: decision:…/inline-registry-table; it belongs to the allocation
                                               write, which epic:parallel-workspaces/environment-lifecycle builds
a name is allocated                         -> the file is rewritten on every allocation, also when the name is recorded
a boolean value under [env_index]           -> read as the reference tool reads it: `true` as the index 1 and `false`
                                               as the index 0, its test of an integer accepting a boolean; to be
                                               confirmed by a probe (Open questions)
```

`ws index` is the one command of this epic that reads the file; the writes are built with
`epic:parallel-workspaces/environment-lifecycle`.

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

An alias listed twice in `env_aliases` resolves to its later position. An `envs_per_workspace` above the largest
unsigned 32-bit value is used as that value: the hash has 65536 values, so every band wider than that gives the same
index.

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
[results-and-exit-codes.md](../../component-runtime/spec/results-and-exit-codes.md).

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
[invocation contract](../../../../work/agent-ready-operation/executable-extensions/spec/invocation-contract.md).

## Open questions

- What `state.toml` holds after its last name is removed — a bare `[env_index]` header or an empty file — is not
  confirmed from source. Engineering records it with the first `ws destroy` scenario.
- That a boolean under `[env_index]`, such as `feature-x = true`, is read as the index one by the reference tool is to
  be confirmed by a probe: the recorded scenario `ws-index/registry-boolean-value`. The binary reads the value as the
  record shows the reference tool reads it.
- That the reference tool exits 1 on a `state.toml` that is not UTF-8, on a directory at its path, and on a key defined
  twice or over a key that holds a value in the body of a table section or inside an inline table, and reads as empty a
  syntax error, a key defined twice above the first table header, and a table header given twice, is read in its source
  and to be confirmed by a probe: the recorded scenarios `ws-index/state-not-utf8`, `ws-index/state-is-directory`,
  `ws-index/state-duplicate-key`, `ws-index/state-duplicate-inline-key`, `ws-index/state-subtable-over-value`,
  `ws-index/state-dotted-key-over-value`, `ws-index/state-duplicate-dotted-key`, `ws-index/state-invalid-toml`,
  `ws-index/state-duplicate-top-level-key`, and `ws-index/state-duplicate-table-header`.
- What a lock entry whose field is not a string reads as in the reference tool is to be confirmed by a probe when
  `epic:parallel-workspaces/reproducible-inputs` first reads the lock.
