---
name: host-scope-cache
depends-on:
  - hub-scope-cache
  - epic:config/runtime
code:
  - blizzard/src/blizzard/runner/composition.py::HostRunnerSet
  - blizzard/src/blizzard/runner/composition.py::start_runner
  - blizzard/src/blizzard/runner/composition.py::drain_runner
---

# Host translation and runner child ownership

Builds the host translation and `HostRunnerSet`'s ownership of runner children in spec
[Scope ownership](../spec/scopes.md), delivering `engineering:locked-child-container`, `engineering:scope-identity`,
`engineering:container-per-process` and the host portion of `engineering:ordered-teardown`. Follows
`decision:adopt-di/cached-scopes`; production activation remains with `host-scope-pilot`.

## Change

- Add the host translation, mapping `PROCESS`, `RUNNER` and `OPERATION`, and the host root that registers
  `HostRunnerSet` at host lifetime.
- Make `HostRunnerSet` the only owner of runner children ([Flows](../spec/scopes.md#flows)). Each child opens through
  `open_child` with the runner's context values (its runner id and `BuiltRunner`), and its owners build their first
  object when it opens.
- Implement the closure paths at the spec's order: drain closes the runner child, then the wiring; `abandon` adopts the
  same order; a failed start before placement closes the child; host stop (`HostRunnerSet.stop_all`) closes, per runner,
  its child and then its wiring, in place of today's wiring-first `close_all`, before the host's own resources.
- The runner child closes its executor, stepper and per-runner hub clients in reverse construction order; closing the
  wiring closes only what the served runner holds outside the child.
- Keep the running host on its existing wiring; exercise child ownership with test-owned graph inputs. The close-order
  change is live in this slice: `HostRunnerSet.abandon` and host stop's `close_all` take the drain order for today's
  runners too, closing the runner graph (`close_runner`) before the wiring.

## Acceptance

- `blizzard:unit-test`: two runner children share only host-owned resources; thirty-two first resolutions do not
  duplicate a runner service; a lookup through a closed child raises `ScopeClosed`.
- `blizzard:unit-test`: drain and `abandon` close the child, then the wiring; host stop closes every child, then its
  wiring, before the host's own resources; each path closes a child exactly once; a failed child publishes nothing and
  the next attempt opens a new one.
- `blizzard:unit-test`: the host translation rejects `TENANT`, `REQUEST` and `COMMAND`; borrowed inputs stay open.
- `blizzard:component-test`: on the running host, `abandon` after a failed placement and host stop both close each
  runner's graph before its wiring, and drain is unchanged; `blizzard:typecheck` passes.
