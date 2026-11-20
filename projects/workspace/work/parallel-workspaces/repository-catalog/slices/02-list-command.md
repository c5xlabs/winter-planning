---
name: list-command
depends-on:
  - epic:parallel-workspaces/workspace-configuration
code:
  - winter-workspace/crates/core/src/diagnostics.rs
  - winter-workspace/crates/core/src/files.rs::ReadFiles
  - winter-workspace/crates/core/src/files.rs::OsFiles
  - winter-workspace/crates/core/src/files.rs::MemoryFiles
---

# `repo list`, the drift warning, and the `Diagnostics` borrow

Adds the `repo list` command of [repository-list.md](../spec/repository-list.md) and the drift module of
[drift-warning.md](../spec/drift-warning.md), with the parts of the seam they are the first to need: `exists`, `is_dir`,
and `entries` of `ReadFiles` (drift-warning.md, Domain), and `Diagnostics` with `diagnostics::notice`
(repository-list.md, Domain). The repository entries come from the loaded `Workspace` as they are. Delivers the list and
implicit-repository bullets of `product:repository-roles`, the drift-warning bullet and the repository-list half of the
creates-nothing bullet of `product:inventory-presence`, under
`decision:parallel-workspaces/repository-catalog/repository-identities` and
`decision:parallel-workspaces/workspace-configuration/domain-reporting`. The drift module is the shared contract the
`ws` commands of other epics call; none of them is wired here.

## Change

- `ReadFiles` gains `exists`, `is_dir`, and `entries` on `OsFiles`, following symbolic links, and on `MemoryFiles`,
  which becomes the model drift-warning.md, Domain, states: per path a file, a directory, a symbolic link, or an entry
  failing reads. `std::fs` stays inside `files.rs`.
- `crates/core/src/drift.rs` holds `Drift`, `detect`, and `notice_line`: `<root>/projects` listed through `entries` and
  `is_dir`, dot-names left out, names compared as bytes, `missing` and `undeclared` sorted, a declared name twice listed
  twice; a listing failure is `InternalError::Unexpected`.
- `crates/core/src/diagnostics.rs` gains `Diagnostics { err, log }` and `notice`, as repository-list.md, Domain, and
  [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Diagnostics on the stderr
  handle, give them; `repo list` is the first caller of both.
- `crates/core/src/commands/repo/list.rs` registers `repo list`, builds `Diagnostics` from `cx.err` and `cx.log`, and
  makes its one call. `repository_catalog::list` runs `config_load::load`, then returns the singletons (`product` when
  `<root>/product` is a directory, `harness` when `<root>/context/harness/.git` exists), then
  `workspace.config.standalone_repositories`, then `workspace.config.project_repositories`, with the locations of the
  spec, normalised as pathlib joins. When there is drift it writes the notice through `Diagnostics` before it returns,
  under `--json` as well.
- The command file writes the spec's Shape A array, or one human row per entry with the URL or `-`, the type, the
  root-relative location, and a pinned mark, through `output`; nothing for an empty human inventory. It exits 0 and
  writes no file.
- `context/architecture/structure.md` and `changes.md` gain the rows for `drift.rs`, the command file, `Diagnostics`,
  and the new `files.rs` content; `changes.md`, A command, states that a command file may build `Diagnostics` from
  `cx.err` and `cx.log` and hand it to its one call.
- `drift` joins `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs`, as `changes.md`, A command, requires of a
  module only a domain entry calls; the pinned test of the list is renamed for its new count, nineteen.
- Recorded scenarios under `scenarios/repo-list/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task, `stdout = "json"`: an empty inventory; projects from both files with one pinned,
  one `name = ""` with a url, and one duplicate name; a standalone with and without `path`; `product/` from `files/` and
  `context/harness/.git` made by the raw git call `git init context/harness`; a project missing from `projects/` and an
  undeclared directory there, with a dot-directory beside it; the same outside a workspace.

## Acceptance

- `cargo scenarios repo-list` passes every scenario; the drift scenario's stored `stderr` names the missing and the
  undeclared names.
- Unit tests of `repository_catalog::list` with `MemoryFiles`: two shared and one local project entry come out in that
  order; an entry with neither `name` nor `url` is absent; `name = ""` with a url is listed by its URL-derived name; two
  projects of one resolved name are both listed; `pinned = 1` lists as pinned; a standalone with `path = ""` is listed
  at `<root>/<name>`; with drift, the notice is in the `Diagnostics` buffer and nothing else is written there.
- Unit tests of `files.rs` inside a `TempTree`: `exists` and `is_dir` on a file, a directory, a symbolic link to each,
  and a dangling link; `entries` of a directory holding a file, a directory, and a dot-entry returns all three;
  `entries` of a missing directory is an error. The same cases on `MemoryFiles`.
- Unit tests of `detect` and `notice_line` with `MemoryFiles`: no `projects/` gives no drift; a file under `projects/`
  is not a directory; `.hidden/` is left out; an entry failing reads at `projects/` gives `InternalError::Unexpected`;
  with missing `b`, `a` and undeclared `c`, the line holds `a` before `b` and both before `c`; no drift gives no line.
- Component tests: the notice is on stderr and absent from stdout under `--json`; a drift-free workspace writes nothing
  to stderr; the tree is unchanged after every run.
- `cargo dev gate full` exits 0.

## Out of scope

- Writing the drift notice from `ws list`, `ws connect`, `ws disconnect`, `ws diff`, `ws fetch`, `ws pull`, `ws merge`,
  `ws push`, `ws checkout`, `ws reset`, `ws clean`, and `ws update`: the epics drift-warning.md names as wiring each.
- `resolve`: `standalone-location`.
