---
name: repository-entries
depends-on:
  - layered-load
  - top-level-refusals
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
  `RepoRef`, and `RepositoryFault` as the spec's Domain gives them; `ValueType` gains `owner`, carrying the repository
  for a repository field.
- The `project_repository` array and its entries are checked, then the `standalone_repository` array and its entries,
  each in declaration order. An array value the reference fails on is `ValueType`; any other non-array reads as no
  entries, as its Value types row says. An element that is not a table, and an entry whose `name` and `url` are both
  empty, is skipped with no message. The resolved name follows the spec's Repository entries, using `config_edit`'s
  existing URL derivation. Existing tests this contradicts are rewritten, not taken as evidence against the refusal:
  `the_document_holds_the_name_as_the_target_gives_it_not_as_the_block_does` (`name = 7`). The second Open-questions
  bullet of `repository-removal.md` is updated, since the load now refuses a non-string `name` before removal.
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
- Unit tests of the arrays: `project_repository = true` and `standalone_repository = 1` are `ValueType`;
  `project_repository = false`, `= ""`, `= "abc"`, and `= { a = 1 }` load with no entries.
- Unit tests: two standalones named by URL `…/ctx.git` and `name = "ctx"` are `DuplicateStandaloneName`; two projects
  named `web` load; `prefix = "ws"` on a standalone, with the default workspace prefix, is `PrefixCollision`; a string
  element of `project_repository` and a table with neither `name` nor `url`, and `{ name = "" }`, are skipped;
  standalones with `url = "host:ctx.git"` and `name = "ctx"` are `DuplicateStandaloneName`.
- A unit test with a faulty project entry and a faulty standalone entry reports the project entry.
- A component test of `ws index` with an absolute standalone `path` exits 1 with a diagnostic naming the standalone.
- `cargo dev gate full` exits 0.

## Out of scope

- `repo add` and `repo list`: `epic:parallel-workspaces/repository-catalog`.
- `config_dir(ext)`: built with its first reader, under the spec's Derived values.
