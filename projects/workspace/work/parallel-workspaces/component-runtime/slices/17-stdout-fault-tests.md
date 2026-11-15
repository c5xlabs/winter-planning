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
  deadline, the process group, and the reaping of `invoke`; stderr is captured as `invoke` captures it. The runner sets
  no signal disposition for the child ([test-harness.md](../spec/test-harness.md), Interfaces, "signals"). The
  documentation of each variant says what file descriptor 1 is when the child starts and, where a write does something
  else to the built binary than to another child, both ("a faulted stdout" in the same place).
- `crates/cli/tests/closed_stdout.rs` exists and runs the built binary through it.
- `context/standards/testing.md`:
  - the executable tier names the three faults and says they are the one thing that tier holds for a command beyond its
    recorded scenarios;
  - in What a test never does, the item that begins "It starts a child only through" names `invoke_with_stdout` beside
    `invoke` and `Git`: its first sentence would otherwise be false of the new test file. Nothing else of the item
    changes;
  - the sentence "A test of the support package that needs a child with a behaviour of its own runs `/bin/sh` as that
    child" stays the rule, and the support package's tests of this change follow it: the child that writes to stdout is
    `/bin/sh`;
  - the section The process runner is left as it is: it describes `invoke`, every sentence of it stays true, and the
    Tiers item is where `invoke_with_stdout` is introduced.
  - Regions shared with `interrupt`, which edits the same file: this change edits only the item of the Tiers list that
    begins "An executable test starts the binary" and the item of What a test never does that begins "It starts a child
    only through"; `interrupt` edits the item that begins "A unit test takes no file", the section Tests that must fail,
    and at most the Unit row of the Tiers table. The two sets share no line. Whichever of the two merges second rebases.
- Files touched: `crates/test-support/src/{process,lib}.rs`, `crates/test-support/tests/process.rs`,
  `crates/cli/tests/closed_stdout.rs` (new), `context/standards/testing.md` in the two items named above. No manifest
  changes: `libc` is already a dependency of the support package.
- `crates/dev/tests/architecture.rs` pins nothing this change alters. The new test file and the support package's source
  hold neither the reference variable's name nor the word the support package never spells
  ([test-harness.md](../spec/test-harness.md), The pinned environment).
- Scenarios: none; the scenario format has no step that takes stdout away.

## Acceptance

- Tests of the support package in `crates/test-support/tests/process.rs`, the package's integration-test target for the
  process runner and no unit tests, since a unit test starts no process; each runs `/bin/sh` as a child that writes one
  line to stdout and then exits with a code that tells whether the write succeeded, and each reaps the child within the
  deadline. `/bin/sh` is not the built binary, so each asserts what a child without the Rust runtime observes
  ([test-harness.md](../spec/test-harness.md), Interfaces, "a faulted stdout"):
  - `StdoutFault::ClosedPipe`, the script opening with `trap '' PIPE`: the write fails, and the child ends by its exit
    code and not by a signal;
  - `StdoutFault::ClosedPipe`, the same script without the `trap`: the child is ended by SIGPIPE, `Exit::Signal`, which
    shows the runner leaves the signal at its default;
  - `StdoutFault::Full`: the write fails;
  - `StdoutFault::Closed`: the write is refused, the child having no file descriptor 1; the null device in its place
    would have accepted it.
- `crates/cli/tests/closed_stdout.rs`, each in an isolated tree; the binary's own runtime is what makes `ClosedPipe` an
  exit code and `Closed` a success, and no test sets a signal disposition or a descriptor for it:
  - `StdoutFault::ClosedPipe` with `--help`, `--version`, and `ws index NAME --json`: exit 1, stderr empty;
  - `StdoutFault::Full` with the same three: exit 1, stderr one error line;
  - `StdoutFault::Closed` with the same three: exit 0, stderr empty;
  - `repo remove` under each fault: the configuration file holds the edit afterwards, and the exit code is the one
    above.
- `cargo dev gate full` exits 0.

## Out of scope

- A failing stderr at the process level: [runtime.md](../spec/runtime.md) holds it by the unit tests of the boundary.
- A stdin that is closed or a terminal: no present command reads stdin.
