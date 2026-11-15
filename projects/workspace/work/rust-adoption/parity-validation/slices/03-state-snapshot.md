---
name: state-snapshot
depends-on:
  - fixture-builder
---

# State snapshot and root normalization

Builds `scenario::snapshot` and `scenario::normalize`: the State snapshot and Normalization sections of
[scenario-format.md](../spec/scenario-format.md). Delivers the stored-state clause of the first acceptance bullet of
`engineering:behavioral-fixtures` and the root-path bullet of `engineering:pinned-scenario-environment`.

## Change

- A snapshot of a tree yields `tree.txt`, `git.txt`, and the changed-file set against a previous snapshot, with the
  exclusions the spec lists.
- Root normalization replaces the canonical root path with `<ROOT>` in bytes, before anything is parsed; the content
  hash of a `tree.txt` line is taken over the content as stored, per the spec's hash rule.
- A JSON stream is parsed and re-encoded canonically for `stdout.json`, in both `json` and `json-lines` modes.
- Snapshots are written to and read from the `before/` and `after/` directory layout, with the spec's stored-path
  encoding; a read is given the snapshot before it and fails when a file `tree.txt` implies was stored is absent.
- Files touched: `crates/test-support/src/scenario.rs`, `crates/test-support/src/scenario/{snapshot,normalize}.rs`,
  `crates/test-support/tests/scenario_snapshot.rs`.

## Acceptance

- Snapshots of the same built workspace in two trees are byte-equal after normalization, including the `tree.txt` hash
  of a file whose content holds the tree's root path.
- A file that is not valid UTF-8 is stored raw and hashed raw.
- `tree.txt` lists a directory, an executable file, a non-executable file, and a symbolic link in the spec's line forms,
  sorted, and lists nothing under `bin/`, `calls/`, `cache/`, `tmp/`, `git-template/`, or inside a git directory.
- A repository under `cache/` has no section in `git.txt`, and nothing beneath `cache/` is stored.
- After one file is modified and one created, the second snapshot's file set holds exactly those two.
- `git.txt` for a workspace with a main checkout, a worktree, and a bare remote has three sections with the spec's
  blocks, the bare one without `dot-git`, `status`, and `stash`; a commit landing in the bare remote changes, in the
  remote's section, `head`, `refs`, and `log` and no other block; a commit made in a work tree changes those and the
  `status` and `worktrees` blocks of the repositories that show it, and nothing else; a reflog-only change changes
  nothing.
- A key listed in `git_config_ignore` is absent from the `config` block.
- Normalizing a buffer that holds the root path twice yields two `<ROOT>` tokens and no remainder of the path.
- A `json` stream of two concatenated documents yields two lines with sorted keys; under `json-lines` a non-JSON line
  becomes a `$text` item.
- A snapshot written to disk and read back equals the original.
- Files named `.git`, `.gitignore`, `.gitattributes`, and `50%.txt` are written under `files/` with the spec's stored
  names and read back under their tree paths; `tree.txt` holds the tree paths; no other segment is renamed.
- After `git add` of its directory in a scratch repository, `git ls-files` lists every file of a written snapshot that
  stores a worktree's `.git` file and a `.gitignore` naming a sibling file.
- Reading a snapshot fails, naming the path, when a file its `tree.txt` implies was stored is absent, and when a stored
  segment is not in the stored form.
- `cargo dev gate slice` exits 0.

## Out of scope

- Deciding whether two snapshots are equal under a scenario's rules: `comparison`.
