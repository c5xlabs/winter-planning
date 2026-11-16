---
name: config-error-variants
depends-on:
  - epic:parallel-workspaces/component-runtime
---

# Every configuration failure has its variant

Adds the variants and carried types of [configuration-model.md](../spec/configuration-model.md), Domain, to
`ConfigError`, so that no later change of this epic edits the shared error type. Delivers the type half of
`engineering:typed-configuration-failures`, and the missing-workspace message of `product:identify-workspace`.

## Change

- `crates/core/src/termination.rs`: `ConfigError` gains `FileWrite`, `RemovedKey`, `InvalidChoice`,
  `DuplicateStandaloneName`, `PrefixCollision`, `RepositoriesNotArray`, `Repository`, `ValueType`, `EnvVar`,
  `ModelTier`, `AgentOverridesNotTable`, and `AgentOverride`, with the carried types `RemovedKey`, `ChoiceKey`,
  `RepoKind`, `EntryIdent`, `RepoRef`, `TypedKey`, `MergedTable`, `ConfigFileName`, `RepositoryFault` and the key enums
  it names, `EnvVarFault`, `ModelTierFault`, `AgentOverrideFault`, `ConfigValue`, and `ValueType`, each spelled as the
  Domain spells it. `ProvisionEntry`, `ServiceEntry`, and `Lock` are not added.
- The three key enums `RepositoryFault` names, which the Domain gives by the keys they stand for: `PathField` has the
  members `Path` and `ConfigDir`; `NestedOnlyKey` has `Envs` and `InheritLocal`; `NestedExcludedKey` has `Load` and
  `Entry`. Each renders as the key's name in the file: `path`, `config_dir`, `envs`, `inherit_local`, `load`, `entry`.
- Every new variant renders a message that names each value it carries. Every `match` on `ConfigError` in the workspace
  stays exhaustive.
- The message of `RootNotFound` names the file looked for, `.winter/config.toml`, beside the directory the search
  started from, as Root discovery of the spec states; the variant keeps its one field. The other existing variants and
  their messages are unchanged.
- `crates/core/src/lib.rs`: the facade exports the new public types beside `ConfigError`.
- Files touched: `crates/core/src/termination.rs`, `crates/core/src/lib.rs`.

## Acceptance

- A unit test in `termination.rs` builds one value of every new variant and asserts that its rendered message contains
  each carried value and that `Termination::Config` of it has exit code 1.
- A unit test asserts that the message of an `EnvVar` variant does not contain a value it was not given.
- A unit test asserts that the message of `RootNotFound { start }` contains the start directory and the text
  `.winter/config.toml`.
- `cargo dev gate slice` passes; no file outside the two named changes.

## Out of scope

- Constructing any new variant outside tests.
