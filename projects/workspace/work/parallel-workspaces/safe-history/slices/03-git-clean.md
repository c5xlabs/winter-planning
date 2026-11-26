---
name: git-clean
depends-on:
  - module-scaffold
  - epic:parallel-workspaces/coordinated-git
---

# Git's clean, listed and run

Adds the `list`, `remove`, and parse of [clean.md](../spec/clean.md), Git's clean, in `history/clean.rs`:
`git clean -nd` and `git clean -fd` in a selected worktree under a C locale, their output read into the removal units
Git reports, a strict parse on success and a lenient one after a non-zero `remove`, which keeps every path already gone.
Never `-x` or `-X`, so ignored files stay. Delivers `engineering:clean-partial-accounting` and the selection-semantics
and ignored-file bullets of `product:remove-untracked-content`.

## Change

- `crates/core/src/history/clean.rs` gains `list`, `remove`, `Removed`, `parse`, and `Parse` of clean.md, Domain; `list`
  and `remove` are each one call through the program port with the span `git clean` and the environment `LC_ALL=C`,
  `LANGUAGE=C`, `LC_MESSAGES=C`; a failure is the repo error with `Clean`.
- The parse keeps each recognised path exactly as Git prints it, quoting included, a directory with its trailing `/`;
  under the strict parse an output with no path and a line it cannot place is a parse failure carrying that diagnosis.
- A worktree path that is no repository is the repo error `NotAGitWorktree`, and a strict parse failure the repo error
  `UnrecognisedClean`.
- `list`, `remove`, `Removed`, and `Parse` are re-exported by name from `lib.rs` for its tests, and `parse` as
  `parse_clean`, since `targets::parse` holds `parse` at the crate root ([clean.md](../spec/clean.md), Domain); no
  command calls them yet.

## Acceptance

- Component tests with real git: `list` names an untracked file and an empty untracked directory, and neither an ignored
  file nor a nested repository; `remove` deletes what `list` named and leaves the ignored file, the nested repository,
  and a tracked modification in place.
- A component test: a file name holding a space and one holding a non-ASCII character are each one path, kept as Git
  quotes it, in both `list` and `remove`.
- A component test: with `LANG` and `LANGUAGE` set to a translated locale in the caller's environment, the output is
  still parsed.
- A component test: a `remove` that exits non-zero after deleting some paths, forced by an untracked directory without
  write permission, answers the paths it removed with the call's failure.
- A component test: `list` and `remove` in a directory that is no repository each fail as the repo error
  `NotAGitWorktree` naming its path, and `remove` reports no path.
- A unit test of the parse: an exit-0 output of one unrecognised line is the parse failure with its diagnosis; "Would
  skip ", "Skipping ", and "warning:" lines are not.
- `cargo dev gate slice` exits 0.
