---
name: repository-entries
depends-on:
  - layered-load
code:
  - winter-workspace/crates/core/src/config.rs
  - winter-workspace/crates/core/src/termination.rs::ConfigError
---

# The field rules of the repository entries

Adds to the load the Repository entries table, the repository rows of Value types, and the repository rows of Cross-key
rules in [configuration-model.md](../spec/configuration-model.md): step 1 of the Load order, and the prefix collision of
step 7. Delivers the duplicate-identity bullet of `product:effective-configuration` and the operation-relevant bullet of
`product:configuration-validation`. Catalog semantics beyond the field rules stay with
`epic:parallel-workspaces/repository-catalog`.

## Change

- `ConfigError` gains `Repository`, `DuplicateStandaloneName`, and `PrefixCollision`, with `RepoKind`, `EntryIdent`,
  `RepoRef`, and `RepositoryFault` as the spec's Domain gives them; `ValueType` carries the repository in `owner` for a
  repository field.
- Project entries are checked, then standalone entries, each in declaration order. An element that is not a table, and
  an entry with neither `name` nor `url`, is skipped with no message. The resolved name is `name`, else the URL's last
  path segment without a trailing `.git`.
- Each field rule of the table is applied, for the kind of entry that has the field: value types for `name`, `url`,
  `main_branch`, `git_excludes`, `cmd`, `ref`, `path`, `prefix`, and `config_dir`; `extension` and `nested` booleans;
  `load` and `entry`; `path` and `config_dir` relative without `..`; the nested-only and nested-excluded keys; a
  standalone with `nested = true`; `envs` and `inherit_local`.
- Two standalone entries resolving to the same name are `DuplicateStandaloneName`; two project entries with the same
  name are accepted. The workspace `prefix` equal to a standalone `prefix` is `PrefixCollision`, at step 7.
- No value is kept.

## Acceptance

- Unit tests, one per `RepositoryFault` variant, each naming the entry's kind and label: an absolute `path`; a `..`
  `config_dir`; `nested = true` on a standalone; `envs` without `nested`; `load` on a nested entry; `extension = true`
  on a nested entry; `extension = "yes"`; `nested = 1`; `load = "soon"`; `entry = []`; `entry = [""]`;
  `entry = ["../x"]`; `envs = 0`; `envs = true`; `inherit_local = ["a.b"]`.
- Unit tests of value types: `cmd = "make"` on a project and `ref = 1` on a standalone are `ValueType` with the
  repository as owner; `cmd = []` loads.
- Unit tests: two standalones named by URL `…/ctx.git` and `name = "ctx"` are `DuplicateStandaloneName`; two projects
  named `web` load; `prefix = "ws"` on a standalone, with the default workspace prefix, is `PrefixCollision`; a string
  element of `project_repository` and a table with neither `name` nor `url` are skipped.
- A unit test with a faulty project entry and a faulty standalone entry reports the project entry.
- A component test of `ws index` with an absolute standalone `path` exits 1 with a diagnostic naming the standalone.
- `cargo dev gate full` exits 0.

## Out of scope

- `repo add`, `repo list`, and what `repo remove` does with an entry the load would refuse:
  `epic:parallel-workspaces/repository-catalog`.
- `config_dir(ext)`: built with its first reader, under the spec's Derived values.
