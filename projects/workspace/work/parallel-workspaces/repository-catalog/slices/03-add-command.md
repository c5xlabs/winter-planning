---
name: add-command
external: github:paul-gross/winter#199
depends-on:
  - remove-refusals
code:
  - winter-workspace/crates/core/src/config_edit.rs::CatalogError
---

# `repo add` of a project or a standalone

Adds the `repo add` command of [repository-addition.md](../spec/repository-addition.md): checks 1 to 5 of the order of
checks, Writing steps a, b, d, e, and f, Block written, Where the block goes, and the result. Delivers the pinning and
dual declaration bullets of `product:repository-roles`; the URL, alias, main-branch, invalid-source, duplicate, and
absolute-or-`..` bullets of `product:repository-addresses`; and the layer, report, duplicate, refused-edits, and
no-clone bullets of `product:edit-repository-inventory`. The write follows `decision:toml-parity` and
`engineering:toml-library`; identities follow `decision:parallel-workspaces/repository-catalog/repository-identities`.
Check 6 and Writing step c follow in later slices, under
`decision:parallel-workspaces/repository-catalog/direct-landing`.

## Change

- `crates/core/src/commands/repo/add.rs` registers `repo add` with the arguments of the `repo add` row of
  [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands.
- `repository_catalog::add` runs `config_load::load`, then checks 2 to 5 in order, against
  `workspace.config.project_repositories` or `standalone_repositories` by `common.url` and `common.name`, each with its
  `CatalogError` variant from the spec's Domain: `InvalidUrl` (the URL shape, Python's `\w` and `\d` as Unicode, one
  final `"\n"` accepted), `PinnedStandalone`, `StandaloneOnly` for `--path`, `--prefix`, then `--ref`, `UnsafePath`, and
  `DuplicateUrl` or `DuplicateName` per existing entry of the kind in merged order, the URL test first.
- The write goes through `WriteFiles::update` on the shared file, or the local file with `--local`, created when absent:
  a missing shared file is `InternalError::Unexpected`; bytes that are not UTF-8 or not valid TOML are
  `ConfigError::FileRead` with `Toml`; an array key holding anything but an array of tables is `NotArrayOfTables`; an
  `Err` of `update` is `ConfigError::FileWrite` with the path of the file; otherwise `config_edit::block_table` builds
  the table of Block written and `append_repository_block` appends it to the array through `toml_edit`, creating the
  array when absent.
- `toml_edit` gains its `display` feature in `crates/core/Cargo.toml`, which serialising the edited document needs;
  `Cargo.lock` and the `toml_edit` row of `context/architecture/structure.md`, The Cargo workspace, follow.
- The command writes the spec's Shape A document under `--json`, else one line naming the kind and the URL; it exits 0.
  It starts no program and writes no other file.
- `context/architecture/structure.md` and `changes.md` gain the rows the new command file and functions need.
- Recorded scenarios under `scenarios/repo-add/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task. Each scenario that writes a file has
  `[[compare.file]] path = "ws/.winter/*.toml" mode = "toml"` with the reason `decision:toml-parity`: a project by URL
  into a file with one block; a standalone with `--path`, `--prefix`, `--ref`, `--main-branch`, two `--git-exclude`, and
  a `--cmd`; `--local` with no local file; `--name ""`; a URL already declared as a project added as a standalone;
  `…/api` beside a declared `…/api.git`; a duplicate URL; a duplicate name; a duplicate of a local entry added to the
  shared file; `--standalone --pinned`; `--path` without `--standalone`; `--path ../x`; `--path=/x`; an invalid URL; an
  invalid URL outside a workspace.

## Acceptance

- `cargo scenarios repo-add` passes every scenario; each refusal's record and replay show exit 1 and the tree unchanged;
  the invalid URL outside a workspace ends as `RootNotFound`.
- Unit tests of the URL shape: the spec's accepted forms, `ssh://user@host/path` refused, a letter of another script in
  the host accepted, one final `"\n"` accepted and two refused.
- Unit tests of `block_table`: each key appears exactly when its condition holds, `--name ""` writes `name = ""`, and an
  empty `--git-exclude` list writes no key.
- Unit tests of `append_repository_block`: the result parses, and its array of kind equals the original's with the block
  appended, every other value equal, for a file with no array, a file with the other array only, a file whose array has
  a sub-table on its last block, and an empty file; `project_repository = []` and an inline array are
  `NotArrayOfTables`; a comment line above an unrelated table and a trailing comment on an unrelated key are still in
  the text after an append, as `toml_edit` keeps them under `decision:toml-parity`.
- Unit tests in `repository_catalog.rs` with a `WriteFiles` stand-in, as its existing update tests use: an update that
  hands the closure no bytes for the shared file ends as an internal failure and writes nothing; an update that fails is
  `FileWrite` with the path of the file.
- Component tests: `project_repository = "abc"` in the shared file, and `[standalone_repository]` in the local file,
  each end `repo add` as `RepositoriesNotArray` from the load, both files byte-equal.
- Component tests: `--local` creates `.winter/config.local.toml` and leaves the shared file byte-equal; the next load of
  every written file succeeds; no `config.lock` or `state.toml` appears.
- `cargo dev gate full` exits 0.

## Out of scope

- Check 6, the standalone location: `standalone-location`. Writing step c: `add-under-lock`.
