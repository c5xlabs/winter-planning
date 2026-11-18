---
name: shared-test-helpers
depends-on:
  - command-span-export
code:
  - winter-workspace/crates/test-support/src/tree.rs::snapshot
  - winter-workspace/crates/core/tests/support/in_process.rs::Ended
  - winter-workspace/context/standards/testing.md
---

# Helpers the component tests share get one owner

Moves the tree comparison three component test files each hold a copy of into the support package, and the assertions
they repeat on an ending onto the scaffold's `Ended`, as [test-harness.md](../spec/test-harness.md), System and
Interfaces, and [runtime.md](../spec/runtime.md), Test seams, give them, under `engineering:testable-boundaries`. No
source under `crates/*/src` of the library or the binary changes, and no test gains or loses an assertion. It is built
before `clock-and-signal-sentences`, which edits `context/standards/testing.md` after it, and may be built beside
`tracing-user-notice`, with which it shares no file.

## Change

- `crates/test-support/src/tree.rs` gains `Found` and `snapshot` as [test-harness.md](../spec/test-harness.md),
  Interfaces, gives them: the code the three test files hold today, made public, with a doc comment on each item.
  `crates/test-support/src/lib.rs` re-exports both by name beside `TempTree`.
- `crates/core/tests/ws_index.rs`, `repo_remove.rs`, and `closed_stdout.rs` drop their own `Found` and `snapshot` and
  import the two from `winter_workspace_test_support`; every call and every `Found::File` they hold stays as it is.
- `crates/core/tests/support/in_process.rs` gains `Ended::document` and `Ended::wrote_nothing` as
  [runtime.md](../spec/runtime.md), Test seams, gives them, and imports `serde_json`, which the library's manifest
  already names.
- The helpers that repeat those two assertions call them and keep what is their own:
  - `ws_index.rs`: `document`, `config_error`, and `usage_parts`, and the three tests that spell them inline;
  - `repo_remove.rs`: `removed` and `catalog_error`;
  - `root.rs`: `usage_error`, when it makes the assertions of `wrote_nothing`, to be confirmed by the builder; when it
    does not, the file is left as it is.
- Each message a helper printed on a failed assertion is kept through the `subject` argument.
- `context/standards/testing.md`: the component tier names `Ended::document` and `Ended::wrote_nothing` beside the
  scaffold, and one sentence says where a helper that more than one test file needs lives: the support package for one
  that reads or builds a tree, `Ended` for an assertion on what `run` answered, and no copy in a test file
  ([contributor-context.md](../spec/contributor-context.md), System).
- Files touched: `crates/test-support/src/{tree,lib}.rs`, `crates/core/tests/support/in_process.rs`,
  `crates/core/tests/{ws_index,repo_remove,closed_stdout,root}.rs`, `context/standards/testing.md`. `root.rs` only on
  the condition above.
- `crates/dev/tests/architecture.rs`: the support package still depends on no workspace package and the library's tests
  already depend on it, so no edge is added and no pin changes; `tree.rs` gains neither forbidden name of the package.
- Scenarios: none.

## Acceptance

- `grep -rn 'fn snapshot\|enum Found' crates/core/tests crates/cli/tests` prints nothing.
- `grep -c 'pub fn snapshot\|pub enum Found' crates/test-support/src/tree.rs` prints 2.
- `grep -n 'Found' crates/test-support/src/lib.rs` prints the re-export line.
- `grep -c 'pub fn document\|pub fn wrote_nothing' crates/core/tests/support/in_process.rs` prints 2.
- `grep -n 'assert_eq!(ended.std' crates/core/tests/ws_index.rs crates/core/tests/repo_remove.rs` prints no line of
  `document`, `removed`, `config_error`, `catalog_error`, or `usage_parts`.
- A unit test in `crates/test-support/src/tree.rs`, in an isolated tree: `snapshot` of a directory that holds a file, a
  directory, and a link that leads outside it answers the three by their paths from it, the link as its target and
  nothing of what it leads to.
- `cargo test -p winter-workspace-core --test '*'` runs the same number of tests as before the change, all passing.
- `git diff --stat` of the change names no file under `crates/core/src` or `crates/cli`, and no manifest.
- `cargo dev gate slice` exits 0.

## Out of scope

- A helper only one test file uses: it stays in that file.
- A writer that fails, which a test keeps in its own file ([runtime.md](../spec/runtime.md), Test seams).
- The signal sentence and the clock sentence of the context documents: `clock-and-signal-sentences`.
