---
name: child-environment
depends-on:
  - module-scaffold
  - epic:parallel-workspaces/workspace-bootstrap
code:
  - winter-workspace/crates/core/src/programs.rs::Program
  - winter-workspace/crates/core/src/programs.rs::find_program
---

# The nested CLI program and its environment

Adds `Program::Winter` and the child environment of [invocation.md](../spec/invocation.md), Domain and Child
environment: the `winter` found on the `PATH` the child receives, the snapshot's chain parsed and extended with this
invocation's resolved root, and the trace context of the active span, all laid over `scrubbed`, which
`epic:parallel-workspaces/workspace-bootstrap` lands for a nested project's `cmd`. Delivers
`engineering:nested-invocation-contract` and `engineering:child-process-isolation` for the environment, and the program
choice of `decision:parallel-workspaces/nested-workspaces/child-compatibility`; `engineering:observability-isolation`
shapes the trace context.

## Change

- `Program` gains `Winter`, named `winter`; its lookup searches the call's `env` overlay `PATH` when it sets one, else
  the snapshot's, so a virtualenv `bin` the scrub dropped is not searched.
- `crates/core/src/nested.rs` gains `CHAIN`, the chain parse of the snapshot's `WINTER_NESTED_CHAIN`, and the child
  environment of a nested CLI call: `scrubbed`'s `remove` and `env`, then `WINTER_NESTED_CHAIN` and, with tracing on,
  `TRACEPARENT` and `TRACESTATE` of the span active at the call.
- `scrubbed` is used as built; a nested project's `cmd` still gets no chain.
- No caller yet.

## Acceptance

- Unit tests over a recording program port: with `PATH` holding `<venv>/bin` and `VIRTUAL_ENV` set to `<venv>/`, the
  `winter` found is the one in a later `PATH` entry, and a `<venv>/bin` spelled with `./` or a trailing `/` is dropped
  too.
- Unit tests of the chain: an absent variable is empty; `a::b:` parses to `a`, `b`; the child's `WINTER_NESTED_CHAIN` is
  the parsed entries then the resolved own root, joined with `:`; an own root reached through a symbolic link is written
  resolved.
- A unit test: the child environment holds no `WINTER_*` name of the snapshot except `WINTER_LOG_LEVEL` and
  `WINTER_OTEL_*`, no `VIRTUAL_ENV`, and no `WINTER_LAUNCH_TIME`, and keeps `LANGUAGE` and `LC_ALL` as the snapshot
  holds them.
- A component test with tracing on: the child's `TRACEPARENT` names the span active at the call, not the snapshot's;
  with tracing off it is the snapshot's.
- `cargo dev gate slice` exits 0.
