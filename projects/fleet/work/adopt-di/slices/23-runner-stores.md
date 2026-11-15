---
name: runner-stores
depends-on:
  - host-infrastructure
code:
  - blizzard/src/blizzard/runner/composition.py::build_stores
  - blizzard/src/blizzard/runner/composition.py::_build_stores
  - blizzard/src/blizzard/runner/composition.py::build_read_stores
---

# Runner-bound repository registrations

Moves the runner-stores row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:one-instance-per-port` under `decision:adopt-di/declared-cutover`.

## Change

- Register bound runner connections and repository/read aliases under each runner child before resolving consumers.
- Keep host stores in the host scope; replace old store constructors with canonical resolutions.
- Narrow affected repository consumers and retain only bounded adapter-internal bundles, not a new service locator.

## Acceptance

- Runner store/isolation tests prove no unbound write or cross-runner read; two runners share the engine, not stores.
- Read/write aliases and remaining runtime references share identity; no second owned store factory survives.
- Strict typing, runner graph/identity guards and relevant component tests pass; configured runner-id behavior is
  unchanged.
