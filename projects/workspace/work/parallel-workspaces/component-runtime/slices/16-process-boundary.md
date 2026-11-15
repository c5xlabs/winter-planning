---
name: process-boundary
depends-on:
  - diagnostics-and-logging
code:
  - winter-workspace/crates/cli/src/main.rs::main
  - winter-workspace/crates/core/src/output.rs
  - winter-workspace/crates/core/src/process.rs::Invocation
---

# The process boundary: panics, the final flush, and an unreadable working directory

Builds `conclude` with its first three parameters, the panic hook, the backtrace rule, `Termination::from_write_error`,
and `Invocation.cwd` of [runtime.md](../spec/runtime.md), The process boundary and What the binary hands over, with the
endings of [results-and-exit-codes.md](../spec/results-and-exit-codes.md), Internal failures and A stdout that cannot be
written, under `engineering:library-output-boundary`,
`decision:parallel-workspaces/component-runtime/internal-failure-rendering`, and
`decision:parallel-workspaces/component-runtime/unwritable-stdout`. It follows `diagnostics-and-logging` and is built
before `stdout-fault-tests` and `interrupt`; it runs beside no other change.

## Change

- `crates/cli/src/main.rs`: `main` is the steps of the spec without the tracer and without the interrupt handler, and
  holds no decision of its own; `conclude` holds the unwind catch, the final flush, the diagnostic, the backtrace, and
  the exit code, over the closures and the handle its caller passes, so unit tests of the file reach every branch
  without a process. The panic hook stores the message and the backtrace in the slot the spec gives, keyed by thread and
  armed by `conclude`, and writes nothing; on a thread that is not armed it hands the panic to the hook that was
  installed before it.
- `termination.rs`: `Termination::from_write_error` exists as the spec gives it. `output.rs` classifies a failed write
  through it and builds an encoding failure through `InternalError::unexpected`; its private classifier and its private
  constructor are gone.
- `process.rs`: `Invocation.cwd` is the `io::Result` of the spec, and step 3 of `run_with` ends as the spec says when it
  is an error and the outcome is `Dispatch`; `run_in_memory` passes a directory of its own. The scaffold's `InProcess`
  gains `cwd_error`.
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `context/architecture/structure.md`: the sentence on `crates/cli/src/main.rs` names `conclude` and the hook.
  - `context/architecture/changes.md`, Writing the result: the sentence "`output.rs` alone decides how a failed write
    ends", and the table under it, name `Termination::from_write_error` as the one classifier, which `output.rs` and the
    binary's final flush both call.
  - `context/standards/rust.md`, Errors: a panic ends as the internal failure, and no code catches an unwind below the
    boundary.
- Files touched: `crates/cli/src/main.rs`, `crates/core/src/{termination,output,process}.rs`,
  `crates/core/tests/support/in_process.rs`, `crates/core/tests/{root,closed_stdout}.rs`, `crates/cli/tests/root.rs`,
  `context/architecture/{structure,changes}.md`, `context/standards/rust.md`, and the scenario directory below. No
  manifest changes. `parse.rs`, `context.rs`, and `lib.rs` are not edited by this change.
- `crates/dev/tests/architecture.rs` pins nothing this change alters. The library token scan reads comments, so no
  comment of `process.rs` spells the working-directory call it stops needing.
- Scenario recorded with `cargo dev record`, exit code alone: `ws-index/outside-workspace`, on a fixture with no
  configuration file at or above the working directory.

## Acceptance

- Unit tests in `crates/cli/src/main.rs`, each calling `conclude` with closures and an in-memory stderr of its own, and
  each installing the hook itself:
  - a run that returns `Success` and a flush that succeeds: code 0, stderr empty;
  - a run that panics: code 1, and the first line of stderr is the error line holding the panic message; the test
    asserts nothing of a backtrace there, because the hook stores `Backtrace::capture()`, which follows the test
    process's `RUST_BACKTRACE`;
  - a run that returns a `Termination::Internal` built with `Backtrace::disabled()`: code 1, one error line, and no
    backtrace; built with `Backtrace::force_capture()`: the error line comes first and the backtrace after it;
  - a panic on a thread no `conclude` armed, while the hook is installed, reaches the previous hook and leaves the slot
    of the armed thread empty;
  - a flush that fails with `BrokenPipe` after `Success`: code 1, stderr empty;
  - a flush that fails with another kind after `Success`: code 1, stderr one error line;
  - a flush that fails after a usage ending: code 2 and the usage diagnostic, unchanged;
  - a stderr whose writes fail: the code the ending gives, and no panic.
- Unit tests of `termination.rs`: `from_write_error` for `BrokenPipe` and for another kind.
- Component tests, through the scaffold: `ws index NAME` with `cwd_error` ends as the spec's ending, exit code 1;
  `--help`, `--version`, and `ws index --bogus` with the same error end as they do without it.
- `grep -n 'unwrap_or_default' crates/cli/src/main.rs` prints nothing.
- `grep -rn 'fn internal' crates/core/src` prints nothing.
- `crates/cli/tests/root.rs`: the built binary run outside any workspace with `ws index NAME` exits 1 with empty stdout.
- The scenario passes, and `cargo dev gate full` exits 0.

## Out of scope

- A real closed pipe, a full device, and a closed descriptor: `stdout-fault-tests`.
- The interrupt handler: `interrupt`.
- The fourth parameter of `conclude` and the span's end and export: `command-span-export`.
