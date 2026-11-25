---
name: port-extensions
depends-on:
  - module-scaffold
code:
  - winter-workspace/crates/core/src/files.rs::WriteFiles
  - winter-workspace/crates/core/src/files.rs::ReadFiles
  - winter-workspace/crates/core/src/programs.rs::RunPrograms
  - winter-workspace/crates/core/src/programs.rs::Call
  - winter-workspace/crates/core/src/programs.rs::Program
  - winter-workspace/crates/core/src/programs.rs::OsPrograms
  - winter-workspace/crates/core/src/process.rs::run
  - winter-workspace/crates/core/src/trace.rs::CommandTracer
  - winter-workspace/crates/cli/src/trace.rs::Tracer
---

# File removal, directory creation, streamed shell commands, and ports shared across threads

Extends the ports for what reconciliation and prune do outside git, and makes them shareable across the fan-out:
`WriteFiles` gains `create_dir_all` and `remove`, as [managed-content.md](../spec/managed-content.md), Domain, gives
them; `ReadFiles`, `WriteFiles`, `RunPrograms`, and `CommandTracer` gain `Sync`, with the tracer change of
managed-content.md, Ports across threads; the program port gains `Program::Shell`, `Stdin`, the `remove` names of
`Call`, and `stream`, as [reconciliation.md](../spec/reconciliation.md), Commands, gives them. Shaped by
`engineering:domain-io-boundaries`, `engineering:process-execution`, and `engineering:observability-isolation`.

## Change

- `OsFiles` implements `create_dir_all` and `remove` with `std::fs`; `remove` removes a symbolic link as a link, a
  directory with everything under it without following links inside it, anything else by unlink, and a missing path is
  Ok. `MemoryFiles` stays a `ReadFiles` alone.
- The four traits gain `Sync`. `crates/cli/src/trace.rs`'s `Tracer` holds its state in a `Mutex`, and `operation` opens
  its span with the command span as explicit parent. The test doubles that hold a `RefCell` hold a `Mutex`.
- `Call` gains `stdin: Stdin` and `remove: &[&OsStr]`; every existing call, the struct literals of
  `crates/core/tests/programs.rs` among them, passes `Stdin::Null` and no names, so every git call behaves as before.
  `remove` names are left out of the child environment before `env` applies.
- `Program::Shell` runs `/bin/sh -c <command>`; `stream` joins the child's stderr to its stdout pipe and hands the
  caller each line as it arrives, the last one without "\n" included, and returns the exit: a code, or the signal. Its
  default body returns an `Unsupported` error, so a `RunPrograms` double that streams nothing needs no change.
- `OsPrograms::new` gains `inherit_stdin`, which `run` takes from its caller: `main.rs` passes true, `InProcess` false.
- `context/architecture/structure.md` records the worker threads of `bootstrap.rs` and the inherited stdin as the
  exceptions managed-content.md and reconciliation.md give.
- No caller uses the new methods yet, apart from tests.

## Acceptance

- Unit tests of `OsFiles::remove` in a temporary tree: a link to a directory outside the tree is removed and its target
  survives; a directory holding such a link is removed and the target survives; a missing path is Ok.
- Component tests in `crates/core/tests/programs.rs` of `stream`: a command writing to stdout and stderr hands both
  lines in order of writing; output without a final newline yields its last line; `exit 3` returns `Code(3)`;
  `kill -TERM $$` returns the signal.
- Component tests: a `Call` whose `remove` names a variable set in the snapshot starts a child that does not see it;
  with `inherit_stdin` false, a `Stdin::Inherit` child reads empty input.
- A unit test of the tracer: an operation span opened on a scoped thread while the command span is open is exported as a
  child of the command span.
- Every existing test passes.
- `cargo dev gate slice` exits 0.
