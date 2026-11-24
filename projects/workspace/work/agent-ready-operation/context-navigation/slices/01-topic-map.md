---
name: topic-map
depends-on:
  - epic:parallel-workspaces/component-runtime
code:
  - winter-workspace/crates/core/build.rs::main
  - winter-workspace/crates/dev/src/fmt.rs::format_files
---

# Compile the topic sources into the binary and expose the eager safety text

Adds the build step that turns `crates/core/topics/**/*.md` into the compiled topic map, the `topics.rs` module that
holds it, the two required topics `root` and `safety`, and `eager_text()` for
`epic:parallel-workspaces/workspace-bootstrap`. Builds the System, Domain (apart from lookup), Reserved ids,
Composition, Eager text for bootstrap, and Topic source sections of [spec.md](../spec.md). Delivers
`engineering:version-bound-context`, `engineering:entry-size`, `engineering:safety-text-consistency`, and
`product:eager-workspace-safety`, under `decision:agent-ready-operation/context-navigation/topic-addressing` and
`decision:agent-ready-operation/context-navigation/safety-hook`.

## Change

- `crates/core/build/topics.rs` holds the rules of spec.md, Topic source, and the code generation as pure functions over
  relative paths and file contents. `build.rs` includes it, walks `crates/core/topics/`, prints `cargo:rerun-if-changed`
  for that directory and every directory under it, and writes `$OUT_DIR/topics.rs`. A violation fails the build with a
  message that starts with the offending path. Under `cfg(test)` the library includes the file too, as it does
  `build/discover.rs`.
- The generated map holds one `Topic` per authored source plus the generated `topics`, sorted by id; each `next` lists
  the referenced ids in order of first reference, without repeats or the topic's own id; the served text of `root` is
  the body of `safety`, a line break, and the body of `root`, as spec.md, Composition, gives it.
- `crates/core/src/topics.rs` includes the generated file and holds `TopicId`, `Topic`, `TOPICS`, and `eager_text()`.
  `lib.rs` declares the module and re-exports `eager_text`, making it a public entry for workspace-bootstrap.
- `crates/core/topics/` holds `safety.md` with the text spec.md, Eager text for bootstrap, gives it, and `root.md`, the
  entry point: what the guidance covers and `winter-workspace context topics`.
- `crates/dev/src/architecture.rs` gains the names spec.md, System, lists, applied to `crates/core/src/topics.rs`, with
  a test in `crates/dev/tests/architecture.rs`.
- `context/architecture/structure.md` gains the rows of `topics/`, `build/topics.rs`, and `src/topics.rs`;
  `context/architecture/changes.md`, What stays a shared edit, gains a topic: a Markdown file under
  `crates/core/topics/`, found by the library's build script, and a reference to it from `root` or from its group's
  topic, since the build refuses a topic `root` does not reach.
- `crates/core/Cargo.toml` adds pulldown-cmark as spec.md, System, gives it, a build- and dev-dependency only.
- `crates/dev/src/fmt.rs` names `crates/core/build/topics.rs` among the files `cargo fmt` does not reach.
- `context/verification/matrix.md`, `workspace:manual-registration`, also applies to a change touching
  `crates/core/build/topics.rs` or `crates/core/topics/`, and gains two cases: a topic added in a new subdirectory and
  referenced from `root` is served by the rebuilt binary's generated map; a `safety.md` of 16 lines stops the build, and
  the first line of the build script's error output starts with `crates/core/topics/safety.md`.
- Nothing serves a topic yet; no command changes. The `Topic` fields only the lookup reads carry an `allow(dead_code)`
  whose reason states the contract, as the registration types of `registry.rs` do.

## Acceptance

- Unit tests of `build/topics.rs`, one per rule of spec.md, Topic source, each failing with a message that starts with
  the offending path: a file that is neither a directory nor `*.md`; front matter that differs from the exact form; an
  id outside the grammar, a repeated id, and an authored `topics`; a missing `root` or `safety`; a reference to an id no
  topic has; a topic other than `safety` that `root` does not reach; a span `winter-workspace context ws/push --json`; a
  `safety` body of 16 lines and a served `root` of 81 lines.
- Unit tests of reference recognition: `winter-workspace context` names `root`; `winter-workspace context ws/push` names
  `ws/push`; a span `winter-workspace context <topic>` and a span inside a fenced or an indented code block name
  nothing; repeats and the topic's own id are left out of `next`, which keeps the order of first reference.
- A unit test of the generated `topics`: its title is `Workspace topics`, and it holds one line per topic, itself
  included, sorted by id, each `` `winter-workspace context
  <id>` `` and its title.
- Unit tests of the composition: `root`'s text starts with `eager_text()`, and `eager_text()` is the body of the source
  whose id is `safety`, byte for byte; moving `safety.md` into a subdirectory changes neither.
- A unit test: the `safety` source holds the provision-before-run rule, the destructive-scope rule, and
  `winter-workspace context topics`.
- The architecture test fails on a `topics.rs` that names `OsFiles`, and on one that names `std::env::temp_dir`, which
  no earlier rule refuses, and passes on the tree.
- `workspace:manual-registration`, with the two topic cases, ends as it states.
- `cargo dev gate slice` exits 0.

## Out of scope

- Looking a topic up and the `context` command: `context-command`.
- Topics for each command: `command-topics`.
- Placing the eager safety text in `AGENTS.md`: `epic:parallel-workspaces/workspace-bootstrap`.
