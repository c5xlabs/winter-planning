---
name: clock-and-signal-sentences
depends-on:
  - shared-test-helpers
code:
  - winter-workspace/context/architecture/structure.md
  - winter-workspace/context/standards/testing.md
  - winter-workspace/context/verification/matrix.md
---

# True sentences on the clock and on signals

Corrects two statements of the context tree that the sources contradict: which files of the binary read the clock, under
`decision:parallel-workspaces/component-runtime/clock-readers`, and what a test sends to another process, as
[test-harness.md](../spec/test-harness.md), Interfaces, and [contributor-context.md](../spec/contributor-context.md),
System, give them, under `engineering:workspace-development-context`. Documents only: no file under `crates/` changes.
It follows `shared-test-helpers`, which edits `context/standards/testing.md` before it, and is built before
`tracing-user-notice`, which edits `context/architecture/structure.md` after it.

## Change

- `context/architecture/structure.md`, The library and the binary: the sentence that has `main.rs` alone read the clock
  names both files that do: `main.rs` for the instant the trace settings are read at, and `trace.rs` for the instant a
  span ends. `main.rs` stays the one file that names the process streams, reads the process environment, and answers the
  exit code. The sentence of `context/standards/rust.md` that the library reads no clock is true and stays.
- Where the start instant of a span with no launch time is read, in `trace.rs` or in the SDK, is to be confirmed by the
  builder: `grep -n 'SystemTime::now\|Instant::now' crates/cli/src/*.rs` lists every read of the sources, and the
  sentence names no file that list does not hold outside a test module.
- `context/standards/testing.md`:
  - Tests that must fail: the sentence that no test sends a signal to another process is replaced by one that says the
    parent sends its child none, with a link to the bullet below.
  - What a test never does gains one bullet, the owner of the rule: a test sends no signal to another process, with the
    two exceptions [test-harness.md](../spec/test-harness.md), `invoke`, signals sent, lists and no other: the runner's
    SIGKILL of the group of the child it started, and the one test of the support package that ends a process that left
    that group. The bullet says that nothing sends SIGINT, and that a test needing a signalled child has it signal
    itself.
  - The sentence of the unit tier that a child raises the signal on itself and the test sends none is true and stays.
- `context/verification/matrix.md`: each place that says no test sends a signal, two as the document stands, says that
  no test sends SIGINT to the binary, and links to the bullet of `testing.md` in place of restating the rule. Every
  method id and every `**Setup.**` and `**Success.**` marker stays.
- Files touched: `context/architecture/structure.md`, `context/standards/testing.md`, `context/verification/matrix.md`.
- `crates/dev/tests/architecture.rs` pins the matrix's method ids and markers, link targets, the documented commands,
  and the scope tokens. The change keeps every id and marker, adds a link that resolves, and adds no command and no
  scope token, so neither that file nor `crates/dev/src/architecture.rs` changes.
- Scenarios: none.

## Acceptance

- `grep -n 'SystemTime::now\|Instant::now' crates/cli/src/*.rs` prints lines of `main.rs` and `trace.rs` alone, and the
  sentence of `context/architecture/structure.md` on the clock names those two files.
- `grep -rn 'No test sends a signal to another process' context/` prints nothing.
- `grep -n 'SIGKILL' context/standards/testing.md` prints a line under What a test never does.
- `grep -n 'testing.md#' context/verification/matrix.md` prints a link to the section of that bullet.
- `grep -rn 'libc::kill' crates/test-support crates/cli crates/core` prints the runner's kill of the group, the one test
  of `crates/test-support/tests/process.rs` that ends an escaped process, and probes with signal 0, and no other send.
- `git diff --stat` of the change names the three documents and no file under `crates/`.
- The architecture test's document checks pass, and `cargo dev gate slice` exits 0.

## Out of scope

- The sentence of `testing.md` on shared helpers: `shared-test-helpers`.
- The records the Modules row of the library's `trace.rs` counts: `tracing-user-notice`.
- Having `conclude` supply the end instant: rejected by `decision:parallel-workspaces/component-runtime/clock-readers`.
