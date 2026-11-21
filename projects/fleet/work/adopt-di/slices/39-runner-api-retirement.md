---
name: runner-api-retirement
depends-on:
  - runner-api-host
  - runner-api-leases
  - runner-api-control
  - runner-api-settings
  - runner-api-telemetry
code:
  - blizzard/src/blizzard/runner/app.py::build_hosted_app
  - blizzard/src/blizzard/runner/app.py::AppRunnerStart
  - blizzard/src/blizzard/runner/app.py::_WiredRunner
  - blizzard/src/blizzard/runner/composition.py::HostProcess
  - blizzard/src/blizzard/runner/composition.py::RunnerProcess
  - blizzard/src/blizzard/runner/composition.py::run_startup_steps
  - blizzard/src/blizzard/runner/api/wiring.py::RunnerWiring
---

# Retire the runner API's legacy owners

Closes runner API/runtime aggregation in spec [Mixed ownership](../spec/migration.md#domain), delivering
`engineering:narrow-api-edge`, `engineering:container-only`, and `engineering:rules-follow-wiring` under
`decision:adopt-di/declared-cutover`.

## Change

- Retire `RunnerWiring` and `ServedRunner.seams` once no route reads them, and the runner app's `app.state` slots that
  only carry a service to a route.
- Classify `LiveRunner`, `StartingRunner`, `AppRunnerStart`, `ResumeMarking` and `_WiredRunner` by the spec's end-state
  table: lifecycle owners and holders stay, with no service forwarding.
- Retain explicit lifecycle/roster owners; remove service aggregation and forwarding from `HostProcess`/`RunnerProcess`.
- First adapt the remaining local runtime consumers, including rejected-tracing announcement in `run_startup_steps` and
  audit/read-repository access in `_reconcile_host_scratch` (also reached per runner through `AppRunnerStart.scratch`),
  to named capabilities and typed factories, so `_reconcile_host_scratch` no longer calls `build_read_stores`; that
  compatibility entrypoint stays for `RunnerStoreFiles.held_bindings` and `RunnerBoundStoreFiles.runner_view`, and
  retires in `cli-runner-writes`. This is consumer narrowing, not the later command-container migration.
- Remove now-empty host/runner legacy registration, bags/assemblers and their allowances; make host guards
  container-only.
- Update host/runner harness guidance and exemplars while retaining remaining CLI transition guidance.

## Acceptance

- `blizzard:structural-gate`: no app/domain/driver consumer takes an all-services runtime bag, and a planted legacy
  construction, bag parameter or `RunnerWiring` read fails its gate; typed driver state is not banned.
- `blizzard:component-test` shutdown tests and `blizzard:typecheck` pass.
- `blizzard:component-test`, `blizzard:service-test` and `blizzard:compat` preserve behavior without client or config
  changes.
- `blizzard-context:reference-lint` and `blizzard-context:registry-drift` pass on the updated host/runner guidance; the
  updated exemplars pass `blizzard:typecheck`, `blizzard:unit-test` and `blizzard:component-test`.
- `blizzard:component-test` (`tests/component/runner/cli`): the still-unconverted host/tick/offline-read command bodies
  pass after aggregate removal; no CLI caller reads a removed runtime field or calls a removed store-bundle constructor.
