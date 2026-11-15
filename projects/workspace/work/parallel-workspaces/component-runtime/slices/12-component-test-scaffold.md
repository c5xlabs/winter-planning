---
name: component-test-scaffold
depends-on:
  - foundation-documents
code:
  - winter-workspace/crates/core/tests/root.rs
  - winter-workspace/crates/core/tests/ws_index.rs
  - winter-workspace/crates/core/tests/repo_remove.rs
  - winter-workspace/crates/core/tests/closed_stdout.rs
---

# One home for the in-process test scaffold

Builds `crates/core/tests/support/in_process.rs` as [runtime.md](../spec/runtime.md), Test seams, gives it, under
`engineering:testable-boundaries`. The component-test files stop carrying their own copy of the code that builds an
`Invocation` over in-memory handles and reads what the run ended with. No production source changes.

## Change

- `crates/core/tests/support/in_process.rs` exists with `InProcess` and `Ended` as the spec gives them, without the
  members the spec marks "with …", and opens with the one lint attribute
  [repository-layout.md](../spec/repository-layout.md), What a command receives, allows it.
- `crates/core/tests/{root,ws_index,repo_remove,closed_stdout}.rs` each declare the module by `#[path]` and build every
  invocation through `InProcess`; their own `Ended` type and `run_with` function are gone. `closed_stdout.rs` keeps its
  failing writer and hands it to `run_with_stdout`: the scaffold holds no writer type. No assertion of those files
  changes.
- The contributor documents name the scaffold:
  - `context/standards/testing.md`: the component tier says an in-process run is built through the scaffold and that a
    test file never builds an `Invocation` itself.
  - `context/architecture/changes.md`: the component-test row of A command points to it.
- Files touched: `crates/core/tests/support/in_process.rs` (new),
  `crates/core/tests/{root,ws_index,repo_remove,closed_stdout}.rs`, `context/standards/testing.md`,
  `context/architecture/changes.md`. No manifest changes.
- `crates/dev/src/architecture.rs` and `crates/dev/tests/architecture.rs` pin nothing this change alters: the library
  token scan reads `crates/core/src/**` only, and Cargo builds no test target from a file in a subdirectory of `tests/`.
- Scenarios: none recorded; the change has no observable behaviour.

## Acceptance

- `grep -rn 'Invocation {' crates/core/tests --include='*.rs'` prints lines of `crates/core/tests/support/in_process.rs`
  only.
- `grep -rnE 'struct Ended|fn run_with' crates/core/tests` prints lines of that file only.
- `grep -rn 'allow(dead_code' crates/core/tests` prints one line, in that file.
- Every component test that passed before passes, with its assertions unmodified.
- `cargo dev gate full` exits 0.

## Out of scope

- A scaffold for executable tests: the process runner of [test-harness.md](../spec/test-harness.md) is one already.
- The stdout faults of a real process: `stdout-fault-tests`.
- Any field of `Invocation` or `Context` the runtime adds, and the scaffold members marked "with …": `root-options`,
  `process-boundary`, `interrupt`, `command-span-export`; each extends the scaffold's one construction site.
