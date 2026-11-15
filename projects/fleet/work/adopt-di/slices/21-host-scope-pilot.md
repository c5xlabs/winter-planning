---
name: host-scope-pilot
depends-on:
  - hub-request-pilot
code:
  - blizzard/src/blizzard/runner/composition.py::build_host_process
  - blizzard/src/blizzard/runner/composition.py::HostProcess
---

# Host and runner child-scope pilot

Activates the host's locked lifetime graph from spec [Scope ownership](../spec/scopes.md), delivering its
`engineering:born-in-container`, `engineering:locked-child-container`, and `engineering:graph-checked-whole`. Uses the
shared opener/ownership mechanism and `decision:adopt-di/cached-scopes`. The hub pilot's executable
authenticated/background proof is the activation prerequisite under `decision:adopt-di/strangler-migration`.

## Change

- Add host translation/root and runner-child ownership, initially retaining named legacy leaves and runtime adapters.
- Register host readiness as a representative area service and consume it through the served app's readiness dependency.
- Land host transitional harness guidance, wiring exemplar and duplicate/identity/lifetime gates.
- Require new host/runner services to use area registration only after the usable pilot lands.

## Acceptance

- Host readiness is served from the new graph; an exemplar/test-only service is registered and consumed through the same
  production resolution path without changing the root's individual service list.
- Two runner children share only host-owned resources; thirty-two first resolutions do not duplicate a runner service.
- Cross-thread and failed-child tests preserve caller-owned values and explicit host/runner closure order.
- Host strict typing, graph/guard tests and relevant readiness/app tests pass with unchanged HTTP behavior.
