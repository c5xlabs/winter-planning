---
name: layered-load
depends-on:
  - epic:parallel-workspaces/component-runtime
code:
  - winter-workspace/crates/core/src/config_raw.rs::load
  - winter-workspace/crates/core/src/workspace_root.rs::find
  - winter-workspace/crates/core/src/index_lookup.rs::answer
---

# The layered load of both configuration files

Builds the Load flow's file reading and the Layering of [configuration-model.md](../spec/configuration-model.md), under
`decision:parallel-workspaces/workspace-configuration/layering-policy`, and the Root discovery diagnostic of the same
spec. Delivers the precedence bullets of `product:effective-configuration`, the missing-versus-corrupt bullet of
`product:configuration-state-ownership`, the selection bullet of `product:identify-workspace`, and
`engineering:inert-configuration-read`. The reading of `env_aliases` and `envs_per_workspace` for `ws index` moves onto
the merged document; every other key stays unread, as the spec's Load section allows until a command reads it.

## Change

- `crates/core/src/config.rs` replaces `config_raw.rs`. It reads `.winter/config.toml` and `.winter/config.local.toml`,
  each missing file as an empty document, and ends as `ConfigError::FileRead` with the `Io` or `Toml` cause for a file
  that exists and cannot be read or parsed, naming that file.
- `[agent_model_overrides]` present and not a table, in either file, is `ConfigError::AgentOverridesNotTable` naming the
  file; the variant is added to `ConfigError` in `termination.rs`.
- One merge function applies the three rules of Layering to the two documents: array append for the two repository
  arrays, a one-level table merge for the seven named tables, and whole replacement for every other key, with each "a
  local value that is not …" case of the rules table. An empty local document returns the shared one unchanged.
- `env_aliases` and `envs_per_workspace` are read, coerced, and defaulted from the merged document as their rows of Top-
  level keys say, and `EnvsPerWorkspaceTooSmall` is checked there; `ws index` answers exactly as before for every
  configuration it accepted before.
- `ConfigError::RootNotFound` names the start directory and the file discovery looks for, `.winter/config.toml`, so the
  diagnostic says how a workspace is selected. Discovery itself is unchanged.
- The load runs no external command and writes nothing; `ws index` loads once per invocation.
- `context/architecture/structure.md`: the `config.rs` row replaces the `config_raw.rs` row.

## Acceptance

- Unit tests of the merge, one per row of the Layering tables: a local repository entry appends after the shared ones; a
  local non-array repository value replaces the shared array; `[git] user = { name = … }` drops the shared `user.email`;
  `[keybindings.bindings]` replaces the shared bindings and keeps `leader` and `timeoutlen`; `[env.feature.vars]`
  replaces every shared feature var and keeps `[env.workspace.vars]`; `[capabilities] service = …` replaces only
  `service`; `[space]` drops the shared `[space.kinds]`; `lint = …` and `git_excludes = …` replace the shared value; a
  local non-table for a merged table leaves the shared table untouched; an empty local document returns the shared
  document unchanged.
- Unit tests of the file reading: a missing local file and a missing shared file each read as empty; an unreadable file
  is `FileRead` with `Io`, and invalid TOML is `FileRead` with `Toml`, each naming the file; `agent_model_overrides = 1`
  in the local file is `AgentOverridesNotTable` naming the local file.
- Component tests of `ws index` in a `WorkspaceBuilder` workspace: `env_aliases` set only in the local file decides the
  alias index; invalid TOML in the local file exits 1 with a diagnostic naming that file; the tree is unchanged after
  each run.
- A component test from a directory with no workspace above it exits 1 with a diagnostic naming the start directory and
  `.winter/config.toml`.
- `grep -rn config_raw crates` prints nothing.
- `cargo dev gate full` exits 0, and the recorded `ws index` scenarios pass unchanged.

## Out of scope

- Refusals of individual keys and tables: `top-level-refusals`, `repository-entries`, `env-bands`, `model-tiers`.
- A command that shows the effective settings: not planned.
