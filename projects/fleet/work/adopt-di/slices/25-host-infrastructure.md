---
name: host-infrastructure
external: github:paul-gross/blizzard#2479
depends-on:
  - host-scope-pilot
code:
  - blizzard/src/blizzard/runner/composition.py::_build_host_graph
  - blizzard/src/blizzard/runner/composition.py::build_host_stores
  - blizzard/src/blizzard/runner/hosting/retirement.py::RetirementNotices
---

# Host-owned infrastructure registrations

Moves the host-infrastructure row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring`, `engineering:one-instance-per-port`, and `engineering:ordered-teardown` under
`decision:adopt-di/declared-cutover`.

## Change

- Register host store adapters through their own area methods, among them the host settings, hosted-runner, hub-token,
  admission, workspace and subscription stores `epic:config/runtime` added. Register the host's engine, which the host
  root owns and closes, with shared clocks/children, identity, readiness, host flags, `Listeners`, `RetirementNotices`,
  telemetry and tracing resources. Settings, records, sealed tokens and hosted runners move in their own slices.
- Register the other members `_build_host_graph` builds today: `HostRegistration`, `HostLaneDrain`, `HttpHostHubClient`
  and the httpx client it owns, `ToolProfileHolder`, `LazyUsageHttpClient`, `HostInvariantCheck` and the host's
  `EventBroker`; and the `HostProcess` members `LinuxProcessProbe`, the host's `zone`, `RunnerStoreConnections`,
  `SpanRateLimiter`, the `ReceiverCounter`s and `ReceiverBounds`, and the trace exporters.
- Register `HostTickInForce` at host lifetime, holding the `HostTick` and the served subscriptions the host is composed
  with, in place of the `HostProcess.host_tick` and `subscriptions` fields; `host_pass_stepper` reads the tick from it
  each pass, and `HostRecordsReconcile` sets it in place of assigning those fields. The container registers no
  `HostTick`, which the reconcile replaces when a subscription record moves
  ([Objects built from settings](../spec/scopes.md#objects-built-from-settings)); `host-records` registers the
  reconcile.
- Replace host legacy construction with canonical values and named runtime collaborators; preserve conditional features.
- Narrow shared consumers as their dependencies move and retire unused host fields/branches.
- Adapt `tools.invariants.runner_invariants` to the registered bounded host-audit capability while preserving its
  callable interface for the existing offline developer command.

## Acceptance

- `blizzard:component-test` host app and tick tests share store and telemetry instances; same-type role clocks remain
  distinct; a subscription record write reaches the next host pass through `HostTickInForce`.
- `blizzard:component-test`: host closure attempts runner closure before disposing its engine and the host hub client's
  httpx client; optional/borrowed resources retain existing owners.
- Explicit factories preserve defaults; `blizzard:typecheck` and `blizzard:unit-test` (the host graph/identity/ownership
  guards in `tests/guards`) pass.
- `blizzard:component-test` host API tests preserve behavior and host-versus-runner store boundaries.
- `blizzard:component-test` (`tests/component/runner/cli`): the existing offline runner invariant command still passes
  without requiring the later command-container migration.
