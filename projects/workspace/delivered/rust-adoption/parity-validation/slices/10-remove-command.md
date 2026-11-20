---
name: remove-command
depends-on:
  - index-command
---

# The first writing command: repo remove, tested in process

Builds `repo remove` as
[repository-removal.md](../../../../work/parallel-workspaces/repository-catalog/spec/repository-removal.md) specifies
it, with the write side of the file port, unit tests, and component tests, and no recorded scenario yet. It is the
command half of the state-changing proof of `engineering:method-proof`, under
`decision:rust-adoption/parity-validation/proof-behaviours`. The block-removal rule is the binary's, under
`decision:parallel-workspaces/repository-catalog/removal-comment-extent`. The reference tool is not run in this change.
It builds on the file port and the `files` field of `Context` that `index-command` adds.

The write port and its first caller arrive together: the library holds no code that nothing uses
([repository-layout.md](../../../../delivered/parallel-workspaces/component-runtime/spec/repository-layout.md),
"warning-free while partly used"), so the port cannot land ahead of the command that calls it.

## Change

- `crates/core/src/commands/repo/remove.rs` is the command file; the `repo` group appears in help.
- `config_edit.rs` holds `remove_repository_block`, `RepositoryKind`, and `CatalogError` as repository-removal.md,
  Domain, gives them. It locates blocks with `toml_edit` and removes a byte range of the original text, so every byte
  outside the range of the spec's What is removed is kept. The effective name of a block follows the spec's three
  numbered cases exactly.
- The write side of the file port arrives: `WriteFiles` and its `OsFiles` implementation in `files.rs`, as
  repository-layout.md, What a command receives, and
  [configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
  Write integrity, give them — the lock, the temporary file and rename, the kept permission bits, and the symlink rule,
  all four; `Context` gains `file_writer`, and `run` binds `OsFiles` to it.
- The test module of `crates/core/src/parse.rs` supplies `file_writer` where it builds a `Context`, and
  `context/architecture/structure.md` lists the new modules.
- `CommandError` gains `Catalog`; `lib.rs` re-exports `CatalogError` and `RepositoryKind` beside `CommandError`.
- The `--json` document is encoded with `serde_json`, which the library already depends on.
- Files touched: `crates/core/src/commands/repo/remove.rs`,
  `crates/core/src/{config_edit,files,context,process,termination,lib}.rs`, `crates/core/src/parse.rs` (its test module
  only, for the `Context` literal that gains the field), `crates/core/Cargo.toml` (`toml_edit`, `libc`), `Cargo.lock`,
  `crates/core/tests/repo_remove.rs`, `context/architecture/structure.md` (for the new modules).

## Acceptance

- Unit tests of `remove_repository_block`, each asserting the whole resulting text:
  - on the commented configuration of [method-proof.md](../spec/method-proof.md), Proof scenarios:
    - removing `web` gives the text of that spec's The deviation scenario, byte for byte: `# second: web` is gone and
      `# third: docs` stands;
    - removing `docs` removes `# third: docs` through the end of the file, and the result ends with the line
      `# after web` followed by one blank line;
    - removing `api` removes its header, its url line, `# after api`, and the blank line after it, and keeps
      `# workspace repositories`, the blank line under it, and `# second: web`;
  - on the plain configuration of the same section, removing `docs` gives a text that ends with the api url line
    followed by one blank line;
  - each of the six rows of the table in repository-removal.md, What is removed, with the Binary column as the expected
    result;
  - blank lines: two blank lines above a block's own comment are both kept; two blank lines between a block's last key
    and the next header's own comment are both removed; a comment run separated from the removed block's header by one
    blank line is kept together with that blank line;
  - the only block of a file; a block matched by its explicit name; a block with neither name nor URL never matches;
  - the effective name, one test per example of repository-removal.md, "effective name of a block": each of the six URLs
    there is matched by the name beside it and by no other; `host/a:b.git` is matched by `b` and not by `a:b`; a block
    with `name = ""` beside a URL is not matched by the URL's name; a block with `name = 7` is matched by `7`; a block
    with `url = ""` and no `name` never matches;
  - two blocks with the same name lose the first only; a `standalone` target leaves a same-named project block alone;
  - a file without a trailing newline keeps that; a comment line indented with a tab counts as a comment; no match
    returns `None`.
- Component tests through `run` with in-memory handles against a `WorkspaceBuilder` workspace:
  - success prints the `repo remove --json` document with `kind` and `name` as given in TARGET, returns
    `Termination::Success`, and changes `.winter/config.toml` and no other file;
  - `repo remove nope` gives `CatalogError::TargetShape`, also from a directory with no workspace above it;
    `repo remove other/x` gives `UnknownKind`; `repo remove project/nope` gives `NotFound` and changes no file; each is
    a `Termination::Command` with exit code 1 and a diagnostic that holds the value at fault;
  - `--local` edits `.winter/config.local.toml` and leaves `.winter/config.toml` byte-equal; `--local` without that file
    gives `NotFound` and creates nothing; a repository declared only in the local file is `NotFound` without `--local`;
  - a configuration file that is not valid TOML gives `ConfigError::FileRead` and is left byte-equal.
- A component test asserts no temporary file remains beside the configuration file after success and after `NotFound`.
- A component test holds the lock of Write integrity from a second handle on another thread: the command has not
  returned after 200 ms, returns `Success` once the lock is released, and the file is then the edited text.
- A component test of the symlink rule: `.winter/config.toml` is a symbolic link to `shared/config.toml` inside the
  workspace. After a successful removal the link is still a symbolic link with the same target text,
  `shared/config.toml` holds the edited text, and neither `.winter/` nor `shared/` holds a temporary file.
- A component test of the permission rule, run twice, with the configuration file's mode set to `0o600` and to `0o640`
  before the command: after a successful removal the file has the mode it had before. One of the two differs from what a
  plain create gives under any umask the test runs with.
- `std::fs` occurs in `crates/core/src` only in `files.rs`; `registry.rs` and `groups.rs` are unchanged, `parse.rs` is
  unchanged outside its test module; no lint attribute was added or removed.
- `cargo dev gate full` exits 0, and every scenario under `scenarios/` still passes.

## Out of scope

- Recording and replaying the command against the reference tool, and the expectation for the deviation:
  `proof-state-changing`.
- `repo add` and `repo list`: `epic:parallel-workspaces/repository-catalog`; cloning:
  `epic:parallel-workspaces/workspace-bootstrap`; lock-file updates: `epic:parallel-workspaces/reproducible-inputs`.
- Every other configuration writer: `epic:parallel-workspaces/workspace-configuration`.
- The effective name of a block whose `name` or `url` has a type the spec's Open questions leaves unpinned.
