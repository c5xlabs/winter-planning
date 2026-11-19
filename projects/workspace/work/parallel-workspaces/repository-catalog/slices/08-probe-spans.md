---
name: probe-spans
depends-on:
  - worktree-status
code:
  - winter-workspace/crates/core/src/trace.rs::CommandTracer
  - winter-workspace/crates/cli/src/trace.rs::Tracer
  - winter-workspace/crates/core/src/context.rs::Context
  - winter-workspace/crates/core/src/process.rs::run_with
  - winter-workspace/crates/core/tests/support/in_process.rs::Recording
---

# One operation span per `--status` probe

Adds the tracing block of [location-listing.md](../spec/location-listing.md), With `--status`, and the tracer-port
extension of [child-processes.md](../spec/child-processes.md), Domain: with tracing on, each probe of
`ws worktrees --status` runs inside a `git status` span that is a child of the command span. Serves
`engineering:observability-isolation`; the command span's contract is
[tracing.md](../../../../delivered/parallel-workspaces/component-runtime/spec/tracing.md), unchanged.

## Change

- `CommandTracer` gains `operation`, with a default body that records nothing, and `OperationSpan` its `failed`;
  `NoTrace` and the scaffold's `Recording` keep the default, and the binary's `Tracer` overrides it: a child of the
  command span, recorded only when the command span is, ended when the handle drops, and sent in the command span's one
  export.
- `Context` gains `tracer`, which `run_with` fills from the `Invocation`, and `locations::list` takes it.
- Each worktree probe opens `git status` with `winter.repo` and `winter.env`, and the workspace probe `git status` with
  `winter.repo` alone, whether or not the probe makes a git call, for every probe that runs; a probe ending in a
  `RepoError` marks its span failed with `error.type` `"RepoError"` and no description. No span is opened without
  `--status`.
- `context/standards/rust.md`, where it describes the `CommandTracer` port, and the `opentelemetry` rows of
  `context/architecture/structure.md` name the operation span beside the command span.

## Acceptance

- Executable tests with the loopback trace collector of the support package: a `--status` run over one environment of
  two projects and the workspace exports four spans, three `git status` children of the command span with the stated
  attributes and no other; a run with a failing worktree probe exports that probe's span with status ERROR and
  `error.type` `"RepoError"`; a run without `--status`, and a run with tracing off, exports no operation span.
- No attribute holds a path, an argument, or git output.
- `cargo dev gate full` exits 0.
