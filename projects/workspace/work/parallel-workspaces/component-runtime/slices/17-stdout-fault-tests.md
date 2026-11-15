---
name: stdout-fault-tests
depends-on:
  - process-boundary
code:
  - winter-workspace/crates/test-support/src/process.rs::invoke
  - winter-workspace/crates/cli/tests
---

# The built binary against a stdout that cannot be written

Builds `StdoutFault` and `invoke_with_stdout` of [test-harness.md](../spec/test-harness.md), Interfaces, and holds the
endings of [results-and-exit-codes.md](../spec/results-and-exit-codes.md), A stdout that cannot be written, at the
executable tier, under `engineering:testable-boundaries` and
`decision:parallel-workspaces/component-runtime/unwritable-stdout`. It follows `process-boundary` and needs no other
change; `interrupt` needs the same one and may be built beside this change, and `runtime-documents` follows both. No
production source changes.

## Change

- `crates/test-support/src/process.rs`: `StdoutFault` and `invoke_with_stdout` exist as the spec gives them, with the
  deadline, the process group, and the reaping of `invoke`; stderr is captured as `invoke` captures it.
- `crates/cli/tests/closed_stdout.rs` exists and runs the built binary through it.
- `context/standards/testing.md`:
  - the executable tier names the three faults and says they are the one thing that tier holds for a command beyond its
    recorded scenarios;
  - the sentence "A test of the support package that needs a child with a behaviour of its own runs `/bin/sh` as that
    child" stays the rule, and the support package's tests of this change follow it: the child that writes to stdout is
    `/bin/sh`.
  - Regions shared with `interrupt`, which edits the same file: this change edits only the item of the Tiers list that
    begins "An executable test starts the binary"; `interrupt` edits the item that begins "A unit test takes no file"
    and the section Tests that must fail. Whichever of the two merges second rebases.
- Files touched: `crates/test-support/src/{process,lib}.rs`, `crates/test-support/tests/process.rs`,
  `crates/cli/tests/closed_stdout.rs` (new), `context/standards/testing.md`. No manifest changes: `libc` is already a
  dependency of the support package.
- `crates/dev/tests/architecture.rs` pins nothing this change alters. The new test file and the support package's source
  hold neither the reference variable's name nor the word the support package never spells
  ([test-harness.md](../spec/test-harness.md), The pinned environment).
- Scenarios: none; the scenario format has no step that takes stdout away.

## Acceptance

- Tests of the support package in `crates/test-support/tests/process.rs`, the package's integration-test target for the
  process runner and no unit tests, since a unit test starts no process; each runs `/bin/sh` as a child that writes one
  line to stdout and then exits with a code that tells whether the write succeeded, and each reaps the child within the
  deadline:
  - `StdoutFault::ClosedPipe`: the write fails, and the child is not ended by a signal;
  - `StdoutFault::Full`: the write fails;
  - `StdoutFault::Closed`: the write succeeds and is dropped, as the spec's comment on the variant says.
- `crates/cli/tests/closed_stdout.rs`, each in an isolated tree:
  - `StdoutFault::ClosedPipe` with `--help`, `--version`, and `ws index NAME --json`: exit 1, stderr empty;
  - `StdoutFault::Full` with the same three: exit 1, stderr one error line;
  - `StdoutFault::Closed` with the same three: exit 0, stderr empty;
  - `repo remove` under each fault: the configuration file holds the edit afterwards, and the exit code is the one
    above.
- `cargo dev gate full` exits 0.

## Out of scope

- A failing stderr at the process level: [runtime.md](../spec/runtime.md) holds it by the unit tests of the boundary.
- A stdin that is closed or a terminal: no present command reads stdin.
