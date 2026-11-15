---
name: runner-loop
depends-on:
  - runner-identity-harnesses
code:
  - blizzard/src/blizzard/runner/loop_wiring.py::LoopWiring
  - blizzard/src/blizzard/runner/loop_wiring.py::RunnerStepper
  - blizzard/src/blizzard/runner/loop/context.py::LoopContext
---

# Container-built loop collaborators

Moves the runner-loop row in spec [Mixed ownership](../spec/migration.md#area-ownership), using operation lifetimes in
[Scope ownership](../spec/scopes.md). Delivers `engineering:narrow-api-edge`'s service-seam portion and
`engineering:container-only` under `decision:adopt-di/declared-cutover`.

## Change

- Register collaborators currently constructed by `LoopWiring.context`/`with_context` through their concept factories.
- Keep typed loop/step contexts, configuration and selected harness snapshots; replace broad store/runtime dependencies
  with the named capabilities their consumers use.
- Supply caller-thread HTTP clients and tracing as borrowed operation values; preserve standalone-versus-borrowed
  runtime ownership and reuse of long-lived context state.
- Remove old loop construction branches and update production/exemplar context wiring together.
- Narrow CLI-hosting driver consumers in `runner/cli/runtime.py`, including runner/pass thread construction, while
  leaving selected-command root/carrier migration for the local CLI slice.

## Acceptance

- Tick/standalone/backfill component tests preserve identity gating, retained state, harness selection and client
  ownership.
- A borrowed runtime stays open after an operation; standalone owned runtime closes; no operation snapshot leaks to a
  parent.
- Step Protocol/narrow-seam guards, graph/identity tests and strict typing pass without passing a container to loop
  steps.
- Existing runner behavior remains unchanged and retired loop bridges are absent.
- Existing host/tick/backfill commands still operate through their old command bodies with the narrowed driver inputs.
