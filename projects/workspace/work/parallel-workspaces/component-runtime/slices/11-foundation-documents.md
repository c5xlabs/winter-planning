---
name: foundation-documents
depends-on:
  - command-file-rule
---

# Where the built binary runs and the README's status

Brings three documents of the code repository in line with the tree they describe, under
`engineering:workspace-development-context`: the built-binary standing rule of
[contributor-context.md](../spec/contributor-context.md), Entry point, in `AGENTS.md` and linked from
`context/tooling/development.md`; and a `README.md` that says what the binary is today. No source file changes. It
follows `command-file-rule` so that it describes the tree that change leaves, and it edits the `context/` tree the two
changes before it edit as well.

## Change

- `AGENTS.md` gains one standing rule, the last of the list, with the content the spec's Entry point gives it:
  - the built binary is run by hand only with its working directory inside an isolated tree, a scratch workspace under a
    temporary directory, or a scratch copy of the checkout under a temporary directory;
  - never in the checkout and never in a real workspace, because the binary resolves the workspace upward from its
    working directory and `repo remove` writes to the configuration it finds there;
  - it is never installed, and never put on `PATH` under the name of the installed `winter`.

  `AGENTS.md` is the rule's one owner. Its one-sentence description of the repository stays.
- `context/tooling/development.md` gains a short section, "Running the built binary", that says where a build puts the
  binary and links to the standing rule in `AGENTS.md` for where it may run. It does not restate the rule and adds no
  fenced command line. It may give the release build's path, `target/release/winter-workspace`, beside the debug
  build's, `target/debug/winter-workspace`: the repository sets no other target directory, so both are where Cargo
  writes the binary.
- `README.md` states the status in its opening paragraph and no longer describes a complete tool:
  - the repository is the Rust source of the `winter-workspace` binary, a work in progress;
  - the commands that exist today are `ws index` and `repo remove`, and `winter-workspace --help` is the reference for
    them. The sentence that names them is worded as a statement of the present — "today" or "so far" — and never as the
    tool's full command set, and it is the same sentence, or the one next to it, that points to `--help` as the
    reference, so a reader of a later tree is sent to the help and not to the list;
  - the binary is not the installed `winter` and does not replace it;
  - the Building section, after the path of the built binary, cautions that `repo remove` edits the workspace
    configuration it finds at or above the working directory, and points to `AGENTS.md` for where the binary may run.

  The Contributing and License sections stay. No feature tour, install step, or command reference is added.
- Files touched: `AGENTS.md`, `context/tooling/development.md`, `README.md`.
- Neither `crates/dev/src/architecture.rs` nor `crates/dev/tests/architecture.rs` is touched: the architecture test
  reads these three files and pins no text this change alters. What it reads of them:
  - the scope tokens of contributor-context.md, System, over all three files. The new text holds none of them: it names
    the installed tool as `winter` and says nothing of that tool's language or tooling.
  - the links of `AGENTS.md` and `development.md`: the link from `development.md` to `../../AGENTS.md` resolves.
  - the fenced command lines of `development.md`, held to the documented commands: none is added.

## Acceptance

- `cargo test -p winter-workspace-dev` passes, and `git diff --stat` of the change lists exactly the three files.
- `grep -c 'PATH' AGENTS.md` prints at least 2, and one bullet of Standing rules holds each of "temporary directory",
  "real workspace", "repo remove", "installed", and "winter".
- `grep -n 'AGENTS.md' context/tooling/development.md` prints a link line under the new heading, and the number of
  fenced blocks in that file is what it was before the change.
- `README.md`:
  - `grep -c 'work in progress' README.md` prints 1;
  - each of `grep -c 'ws index' README.md`, `grep -c 'repo remove' README.md`, and `grep -c -e '--help' README.md`
    prints at least 1;
  - the sentence that names `ws index` and `repo remove` holds "today" or "so far", and that sentence or the one next to
    it names `winter-workspace --help` as the reference;
  - `grep -n 'installed' README.md` prints the sentence that says the binary neither is nor replaces the installed
    `winter`;
  - `grep -n 'feature environments' README.md` prints nothing.
- `cargo dev gate full` exits 0.

## Out of scope

- Restoring a manual, a command reference, or install steps in the README:
  `decision:parallel-workspaces/component-runtime/retained-root-documents`.
- The help texts: the root and group descriptions that read as a full tool, what `(suggested)` means in
  `ws index --help`, and that `repo remove` edits only configuration; the epics that own those commands.
- A check that holds the README's status statement to the command tree: no rule of contributor-context.md covers it.
- The other documents of the context tree: the routing of a new command to recorded scenarios, the rules the
  architecture test holds context documents to, the two routing tables, the package roles of `structure.md`, and the
  restated module-file rule; the rest of this epic.
- "The change is mergeable" in the `workspace:slice-gate` row of the matrix: the rest of this epic.
