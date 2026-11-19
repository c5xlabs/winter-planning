---
name: standalone-location
depends-on:
  - add-command
code:
  - winter-workspace/crates/core/src/files.rs::ReadFiles
  - winter-workspace/crates/core/src/files.rs::OsFiles
  - winter-workspace/crates/core/src/files.rs::MemoryFiles
---

# `repo add --standalone` refuses a location already declared

Adds check 6 of [repository-addition.md](../spec/repository-addition.md), Duplicates, and its first caller's piece of
the seam, `ReadFiles::resolve` ([drift-warning.md](../spec/drift-warning.md), Domain): a new standalone whose location
resolves to the resolved location of a declared standalone is `CatalogError::DuplicatePath`. Delivers the "resolving to
the location of a declared standalone" part of `product:repository-addresses` under
`decision:parallel-workspaces/repository-catalog/location-policy`.

## Change

- `ReadFiles::resolve` on `OsFiles` and `MemoryFiles`, as the `resolve` row of repository-addition.md, Duplicates,
  states it: components left to right, `.` dropped, `..` removing the last component reached, a symbolic link replaced
  by its target read relative to its directory, the rest taken as text from the first component that does not exist.
- After check 5, for kind standalone only, `add` compares `resolve(<root>/(--path when not empty, else the new name))`
  with `resolve(<root>/(its path when not empty, else its resolved name))` of each declared standalone, in merged order;
  the first equal one ends the command as `DuplicatePath` naming `--path` or the new name as given. No other location is
  compared.
- The open question of repository-addition.md on a symbolic-link loop is answered by the scenario recorded here, and the
  answer is folded into the spec's `resolve` row in the same change.
- Recorded scenarios under `scenarios/repo-add/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task, the one that writes a file with
  `[[compare.file]] path = "ws/.winter/*.toml" mode = "toml"` and the reason `decision:toml-parity`: a standalone whose
  default location equals a declared standalone's `path`; a standalone whose `--path` names a declared standalone's
  default location; a standalone at `projects/api` beside a declared project `api`, accepted; a symbolic-link loop on
  the location, the links given in `files/`.

## Acceptance

- `cargo scenarios repo-add` passes, the new scenarios included: the first two exit 1 with the tree unchanged, the third
  exits 0 and writes the block.
- Unit tests of `files.rs` inside a `TempTree`: a path through a symbolic link to a directory resolves to the target;
  `a/../b` gives `b` under the same parent; a path whose middle component does not exist keeps the rest as text; a
  relative link target is read from the link's directory. The same cases on `MemoryFiles`.
- Component tests: a `--path` through a symbolic link to a declared standalone's directory is `DuplicatePath`;
  `--path ext/a` against a declared `ext/a/` is `DuplicatePath`; `--name ..` resolves to the parent of the root and is
  refused only when a declared standalone resolves there too.
- `cargo dev gate full` exits 0.
