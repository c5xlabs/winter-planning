---
name: runner-api-retirement
depends-on:
  - runner-loop
code:
  - blizzard/src/blizzard/runner/app.py::build_hosted_app
  - blizzard/src/blizzard/runner/composition.py::HostProcess
  - blizzard/src/blizzard/runner/composition.py::RunnerProcess
---

# Narrow the runner API and retire legacy owners

Closes runner API/runtime aggregation in spec [Mixed ownership](../spec/migration.md#domain), delivering
`engineering:narrow-api-edge`, `engineering:container-only`, and `engineering:rules-follow-wiring` under
`decision:adopt-di/declared-cutover`.

## Change

- Narrow served-app API, federation, telemetry/readiness and runner-roster selection to named host/runner dependencies.
- Retain explicit lifecycle/roster owners; remove service aggregation and forwarding from `HostProcess`/`RunnerProcess`.
- First adapt the remaining local runtime consumers, including rejected-tracing announcement in `_serve_host` and
  audit/read-repository access in `_reconcile_host_scratch`, to named capabilities and typed factories. This is consumer
  narrowing, not the later command-container migration.
- Remove now-empty host/runner legacy registration, bags/assemblers and their allowances; make host guards
  container-only.
- Update host/runner harness guidance and exemplars while retaining remaining CLI transition guidance.

## Acceptance

- Every runner route resolves at its actual host/runner scope; two-runner API tests preserve isolation and shared host
  data.
- No app/domain/driver consumer takes an all-services runtime bag; typed driver state is not accidentally banned.
- Source/ownership gates reject planted legacy construction or bag parameters; shutdown tests and strict typing pass.
- Relevant component/service and supported-wire tests preserve behavior without requiring client/config changes.
- The still-unconverted host/tick/offline-read command bodies pass their tests after aggregate removal; no CLI caller
  reads a removed runtime field or calls a removed store-bundle constructor.
