# Index lookup

The `ws index` command: what it reads, what it answers, and that it changes nothing. Its arguments are the `ws index`
row of [command-surface.md](../../component-runtime/spec/command-surface.md), Commands. The rule it reports is the
`lookup` line of [lock-and-state.md](../../workspace-configuration/spec/lock-and-state.md), Env index allocation.

## Interfaces

```text
# product:parallel-workspaces/environment-lifecycle/stable-environment-allocation
ws index NAME [--json]
  reads    the workspace root            ../../workspace-configuration/spec/configuration-model.md, Root discovery
           env_aliases, envs_per_workspace, layered and defaulted
                                         same, Load, Layering, Top-level keys, Cross-key rules
           the env index registry        ../../workspace-configuration/spec/lock-and-state.md, `.winter/state.toml`
  writes   nothing: no file under the workspace is created, changed, or locked; a looked-up name is not recorded
  NAME     any string; it need not name an existing environment and is not validated as an environment name
```

| NAME is                        | `index`                                                                                | `source`      |
| ------------------------------ | -------------------------------------------------------------------------------------- | ------------- |
| recorded in the registry       | the recorded value, whatever the aliases now say                                       | `"registry"`  |
| not recorded, and an alias     | the alias's position, `1 ..= N`                                                        | `"suggested"` |
| not recorded, and not an alias | `suggested(NAME)`, the hash slot, whether or not another name is recorded at that slot | `"suggested"` |

```text
no probing  lookup never runs the probe loop of `allocate`. A suggested index is the hash slot alone, so it can equal
            an index the registry records for another name; the index `ws init` later records for NAME may then differ.
            reference: modules/workspace/handlers/workspace_handler.py:796-834, modules/workspace/env_index.py:72-126
recorded    a recorded value is whatever integer the state file holds for NAME — negative, zero, or beyond the band
            included — and is answered unchanged; lookup does not check it against the layout.
            reference: modules/workspace/internal/toml_env_index_registry.py:42-45
--json      one Shape A document (../../component-runtime/spec/output-formats.md, Shape per command):
              {"name": <NAME as given>, "index": <integer>, "source": "registry" | "suggested"}
human       ../../component-runtime/spec/output-formats.md, Human output, row `ws index`; the mark follows the integer
            only for a suggested index of a name that is not an alias
exit        0 for every row above
failures    a missing workspace root and every configuration fault end as the ConfigError of the section that
            defines it. The lookup itself cannot fail: it does not probe, so it never meets a full band, and the
            cross-key rule between the two keys (ConfigError::EnvsPerWorkspaceTooSmall) leaves the band at least
            one slot before lookup runs. EnvIndexError is not part of this command.
```

## Domain

```rust
// crates/core/src/env_index.rs — pure; the formula and its fixed points are lock-and-state.md's
pub(crate) enum IndexSource { Registry, Suggested }
// i64: the range of a TOML integer, so every value a state file can record is answered as recorded.
pub(crate) struct IndexAnswer { pub index: i64, pub source: IndexSource, pub alias: bool }
// Precondition, guaranteed by configuration validation: envs_per_workspace >= aliases.len() + 2.
pub(crate) fn lookup(
    name: &str,
    aliases: &[String],
    envs_per_workspace: u32,
    registry: &std::collections::BTreeMap<String, i64>,
) -> IndexAnswer;
```
