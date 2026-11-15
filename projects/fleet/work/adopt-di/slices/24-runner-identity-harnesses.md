---
name: runner-identity-harnesses
depends-on:
  - runner-stores
code:
  - blizzard/src/blizzard/runner/composition.py::_build_runner
  - blizzard/src/blizzard/runner/composition.py::RunnerProcess
---

# Runner identity and retained harness lifetimes

Moves the runner-identity/harness row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:ordered-teardown`; follows `decision:adopt-di/declared-cutover`.

## Change

- Register runner identity/verdict/token/version/flag holders, clients, executor, retained harness sets and runner
  tracing.
- Preserve dynamic harness-set replacement through the existing owner, rather than freezing its current registry in a
  runner-lifetime service; operation snapshots remain operation-local.
- Replace legacy constructors and narrow moved consumers; keep a runtime owner rather than forwarding `RunnerProcess`.

## Acceptance

- Identity/token/harness tests preserve refresh, refused-token, replacement and retained-invocation behavior.
- Failed runner construction closes acquired clients/executor/sets; normal closure drains work before host disposal.
- Default factories, role keys, strict typing and runner graph/identity gates pass; no borrowed input is closed twice.
- Relevant runner API/component tests pass unchanged and unused runner fields/bridges are removed.
