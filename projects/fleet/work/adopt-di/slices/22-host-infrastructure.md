---
name: host-infrastructure
depends-on:
  - host-scope-pilot
code:
  - blizzard/src/blizzard/runner/composition.py::_build_host_graph
  - blizzard/src/blizzard/runner/composition.py::build_host_stores
---

# Host-owned infrastructure registrations

Moves the host-infrastructure row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring`, `engineering:one-instance-per-port`, and `engineering:ordered-teardown` under
`decision:adopt-di/declared-cutover`.

## Change

- Register host store adapters, workspace/subscription services, shared clocks/children, identity/readiness, host tick,
  telemetry and tracing resources through their own area methods.
- Replace host legacy construction with canonical values and named runtime collaborators; preserve conditional features.
- Narrow shared consumers as their dependencies move and retire unused host fields/branches.
- Adapt `tools.invariants.runner_invariants` to the registered bounded host-audit capability while preserving its
  callable interface for the existing offline developer command.

## Acceptance

- Host app/tick component tests share store/workspace/telemetry instances; same-type role clocks remain distinct.
- Host closure attempts runner closure before disposing its engine; optional/borrowed resources retain existing owners.
- Explicit factories preserve defaults; strict typing and host graph/identity/ownership guards pass.
- Relevant host API/component tests preserve behavior and host-versus-runner store boundaries.
- The existing offline runner invariant command still passes without requiring the later command-container migration.
