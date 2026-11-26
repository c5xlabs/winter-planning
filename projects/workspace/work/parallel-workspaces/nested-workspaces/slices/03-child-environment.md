---
name: child-environment
depends-on:
  - module-scaffold
  - epic:parallel-workspaces/workspace-bootstrap
code:
  - winter-workspace/crates/core/src/programs.rs::Program
  - winter-workspace/crates/core/src/programs.rs::find_program
  - winter-workspace/crates/core/src/bootstrap/checkout.rs::command_environment
  - winter-workspace/crates/core/src/trace.rs::CommandTracer
  - winter-workspace/crates/cli/src/trace.rs::Tracer
---

# The nested CLI program and its environment

Adds `Program::Winter` and the child environment of [invocation.md](../spec/invocation.md), Domain and Child
environment: the `winter` found on the `PATH` the child receives, the snapshot's chain parsed and extended with this
invocation's resolved root, and the trace context of the command span through the new `CommandTracer::child_context`,
all laid over `scrubbed`; and the move of the nested branch of the scrub that
`epic:parallel-workspaces/workspace-bootstrap` landed for a nested project's `cmd` to `nested::scrubbed`, with the
lexical normalise invocation.md, Domain and Child environment, gives. Delivers `engineering:nested-invocation-contract`
and `engineering:child-process-isolation` for the environment, and the program choice of
`decision:parallel-workspaces/nested-workspaces/child-compatibility`; `engineering:observability-isolation` shapes the
trace context.

## Change

- `Program` gains `Winter`, named `winter`; its lookup searches the call's `env` overlay `PATH` when it sets one, else
  the snapshot's, so a virtualenv `bin` the scrub dropped is not searched.
- Only the nested branch of `command_environment` in `crates/core/src/bootstrap/checkout.rs`, the `WINTER_*` and
  `VIRTUAL_ENV` removal and the PATH filter, moves to `crates/core/src/nested.rs` as `scrubbed` and `Scrub`; its PATH
  filter drops every entry whose lexically normalised path equals `normalise(<VIRTUAL_ENV>/bin)` rather than comparing
  bytes. `command_environment` and `CommandEnvironment` stay in `checkout` with the `COMMAND_VARIABLES` overlay and the
  path of a project without `nested`; for a nested project `command_environment` lays `scrubbed`'s `remove` and `env`
  before that overlay, and a `cmd` gets no chain, as before.
- `context/architecture/structure.md`'s bootstrap row gains `nested::scrubbed` among what it may call.
- `crates/core/src/trace.rs`: `CommandTracer` gains `child_context`, with a default body answering `None`, as
  invocation.md, Domain, gives it; `NoTrace` and the test tracers keep the default. `crates/cli/src/trace.rs`: the
  binary's `Tracer` answers the command span's context, injected through `TraceContextPropagator`, and `None` when the
  command span is not recording.
- `crates/core/src/nested.rs` gains `CHAIN`, `chain`, the parse of the snapshot's `WINTER_NESTED_CHAIN`, and
  `child_environment`, the environment of a nested CLI call: `scrubbed`'s `remove` and `env`, then `WINTER_NESTED_CHAIN`
  and the `TRACEPARENT` and `TRACESTATE` of `child_context`, a context with no tracestate putting `TRACESTATE` in
  `remove`.
- `CHAIN`, `child_environment`, and `Scrub` are re-exported by name from `lib.rs` for its tests, and `chain` as
  `nested_chain`, since `epic:parallel-workspaces/stacked-history` has a `chain` of its own
  ([invocation.md](../spec/invocation.md), Domain); no command calls them yet.

## Acceptance

- Unit tests over a recording program port: with `PATH` holding `<venv>/bin` and `VIRTUAL_ENV` set to `<venv>/`, the
  `winter` found is the one in a later `PATH` entry, and a `<venv>/bin` spelled with `./` or a trailing `/` is dropped
  too.
- Unit tests of the chain: an absent variable is empty; `a::b:` parses to `a`, `b`; the child's `WINTER_NESTED_CHAIN` is
  the parsed entries then the resolved own root, joined with `:`; an own root reached through a symbolic link is written
  resolved.
- A component test of `ws init` over a workspace with a nested project whose `cmd` prints `$PATH`, run with
  `VIRTUAL_ENV=<venv>` and `PATH` holding `<venv>/./bin` and `<venv>/bin/` before a later entry: neither spelling
  reaches the `cmd`, the later entry does, and the `cmd` sees no `VIRTUAL_ENV`; a project without `nested` keeps `PATH`
  as given.
- The existing `command_environment` unit tests in `checkout` pass unchanged, and new unit tests of `scrubbed` check the
  normalised spellings `<venv>/./bin` and `<venv>/bin/`.
- A unit test: the child environment holds no `WINTER_*` name of the snapshot except `WINTER_LOG_LEVEL` and
  `WINTER_OTEL_*`, no `VIRTUAL_ENV`, no `WINTER_LAUNCH_TIME`, and none of the `COMMAND_VARIABLES` names, and keeps
  `LANGUAGE` and `LC_ALL` as the snapshot holds them.
- Unit tests over a test tracer: a `child_context` of a traceparent and a tracestate sets both, replacing the
  snapshot's; one with no tracestate sets `TRACEPARENT` and puts `TRACESTATE` in `remove`; `None`, as `NoTrace` answers,
  leaves both as the snapshot holds them.
- A unit test in `crates/cli/src/trace.rs`: with the command span open and recording, the traceparent `child_context`
  answers carries the command span's trace id and span id; with the span not recording it answers `None`.
- `cargo dev gate slice` exits 0.
