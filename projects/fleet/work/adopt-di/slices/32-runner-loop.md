---
name: runner-loop
external: github:paul-gross/blizzard#2486
depends-on:
  - runner-identity-harnesses
code:
  - blizzard/src/blizzard/runner/loop_wiring.py::LoopWiring
  - blizzard/src/blizzard/runner/loop_wiring.py::RunnerStepper
  - blizzard/src/blizzard/runner/loop/context.py::LoopContext
  - blizzard/src/blizzard/runner/loop_wiring.py::RunnerStepper._context
  - blizzard/src/blizzard/runner/app.py::_wire_runner_clients
  - blizzard/src/blizzard/runner/loop_wiring.py::LoopWiring.tick_once
  - blizzard/src/blizzard/runner/loop_wiring.py::PeriodicDriver
---

# Container-built loop collaborators

Moves the runner-loop row in spec [Mixed ownership](../spec/migration.md#area-ownership), using operation lifetimes in
[Scope ownership](../spec/scopes.md). Delivers `engineering:narrow-api-edge`'s service-seam portion and
`engineering:container-only` under `decision:adopt-di/declared-cutover`.

## Change

- Register collaborators currently constructed by `LoopWiring.context`/`with_context` through their concept factories.
- Keep typed loop/step contexts, configuration and selected harness snapshots; replace broad store/runtime dependencies
  with the named capabilities their consumers use.
- Register `RunnerStepper` (today built in `_wire_runner_clients`, `LoopWiring.tick_once` and `PeriodicDriver`) as a
  runner-scoped service that owns the settings-revision context ([Scope ownership](../spec/scopes.md#system)); the
  container supplies its settings reader, harness key and sets, stores, and token holder, and the stepper keeps its
  rebuild, fallback, retention carry-over, and client retirement.
- Supply caller-thread HTTP clients and tracing as borrowed operation values; preserve standalone-versus-borrowed
  runtime ownership and reuse of long-lived context state.
- Remove old loop construction branches and update production/exemplar context wiring together.
- Narrow CLI-hosting driver consumers in `runner/cli/runtime.py`, including runner/pass thread construction, while
  leaving selected-command root/carrier migration for the runner local command slices (`cli-runner-dual-mode`,
  `cli-runner-host`, `cli-runner-writes`, `cli-runner-local`).

## Acceptance

- `blizzard:component-test` tick/standalone/backfill tests preserve identity gating, retained state, harness selection
  and client ownership.
- `blizzard:component-test`: a settings or harness-key revision change rebuilds the context on the next pass and no
  sooner; a failed rebuild keeps the last good context and reports the failed revision; retention state survives a
  rebuild; a replaced hub client closes only once idle; a first build that fails propagates.
- `blizzard:component-test`: a borrowed runtime stays open after an operation; standalone owned runtime closes; no
  operation snapshot leaks to a parent.
- `blizzard:unit-test` step narrow-seam guards (`tests/guards/test_seam_size.py`) and graph/identity guards in
  `tests/guards`, and `blizzard:typecheck`, pass without passing a container to loop steps.
- `blizzard:service-test` runner behavior remains unchanged and retired loop bridges are absent.
- `blizzard:component-test` (`tests/component/runner/cli`): existing host/tick/backfill commands still operate through
  their old command bodies with the narrowed driver inputs.
