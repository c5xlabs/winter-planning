---
initiative: test-suite
refinement: refined
epics:
  - foundation
  - hub-tier
  - runner-tier
  - fleet-tier
  - journey
  - controlled-clock
  - optimization
---

# Test suite

A suite that holds blizzard to its promises from the outside, so a rewritten daemon can be checked against the old one.

## The problem

Someday a daemon will be rewritten, in Python or in something else, and whoever does it will want a suite that tells
them whether the new one still keeps the old one's promises. The service tier is where that suite should live, and today
it cannot serve. A little over a third of its tests describe the hub from the outside, over HTTP. The rest run the
runner inside the test process, step its loop by calling it, and read its store through the ORM, so they can only ever
hold the current Python to its promises. Each test also mints a fixture world, starts a forge, and initializes a hub
through every migration, only to make a handful of requests and throw the whole stack away. The tiers above it share the
same habit: the e2e scenarios tick the runner in-process and read its store, and the journey rehearsal checks its
morning-after state through the Python's own invariant checker.

## The outcome

The service tier, and the tiers above it that describe the platform, move to Playwright Test, written in TypeScript,
reaching blizzard only through what it exposes: its configuration files, its HTTP API and event stream, its CLI, and the
mock fleet's levers. Tests that can share run on a few long-lived hubs per configuration, each with a mock forge and IdP
of its own and shared by a handful of test workers, since one hub is one process and serves one core's worth of tests.
Each test declares the isolation it needs, from a project in a tenant every test shares up to a tenant or a hub of its
own, creates only what it needs inside that, and asserts only what it can see from there. `initiative:multi-tenancy`
built the boundary that makes the sharing possible, and the test suite is the first tenant it was built for.

`epic:test-suite/foundation` brings the Playwright package, the shared-hub harness, the mock fleet changes a shared hub
needs, the blizzard-context changes that redefine the tier, and the three analytics test files ported to prove it.
`epic:test-suite/hub-tier` ports the tests that already speak HTTP and builds the few surfaces they still lack.
`epic:test-suite/runner-tier` brings the real product work the runner's tests need: a host whose loop can be stepped
from outside, and reads of the state its tests check today through the ORM. Tests that start a real runner host move
with it even where their subject is the hub, into `tests/service/fleet/` when the hub is real too.
`epic:test-suite/fleet-tier`, on that host, moves today's e2e scenarios, which are fleet service tests by this
initiative's definition, into `tests/service/fleet/` and the browser scenarios into `tests/ui/`, and retires the e2e
tier until a real work source can be stood up. Each epic builds the public surfaces its own tests need, so the
architecture work arrives just in time rather than in advance.

`epic:test-suite/journey` ports the rehearsal of a whole night to Playwright on daemons of its own, with the consistency
check it asks for answered by the daemons themselves, and beside it the guard that holds `blizzard-mock` to the wire it
imitates. `epic:test-suite/controlled-clock` gives the hub, the runner host, and the mock fleet a clock a test moves
through their surfaces, so a promise about time is proved by advancing past it rather than waiting for it.
`epic:test-suite/optimization` speeds the tiers that stay in Python and the pipeline around them, against the gate the
tier epics leave behind (`decision:test-suite/epic-order`).

## Stance

Independence is the purpose; speed is a guard. The tier never becomes the gate's slowest job, and no port or epic
records the tier's CI time. What a pull request waits on today is the unit and component shards, which
`epic:test-suite/optimization` owns, so a faster service tier alone would not shorten that wait. Where speed and
implementation-independence pull apart, independence wins (`decision:test-suite/black-box-wins`).

Every epic in the initiative keeps these rules:

- New daemon tests are written in Playwright from the day the foundation lands (`decision:test-suite/playwright`,
  `decision:test-suite/one-method`), in the package layout of `decision:test-suite/package-layout`.
- Each test moves once, to its loosest isolation, with its Python original deleted in the same change
  (`decision:test-suite/move-once`).
- Tests seed and assert only through public surfaces (`decision:test-suite/public-surfaces`).
- blizzard-context moves in step with the tests (`decision:test-suite/harness-travels`).
- Nothing is optimized on its way out (`decision:test-suite/no-optimizing-the-doomed`).

The initiative covers every tier above the component tier save one, and the gate's speed. Unit and component tests stay
in Python and stay coupled to it on purpose, and a test whose subject turns out to be internal machinery moves down to
the component tier rather than being ported. The kill-9 crash sweep stays in Python too
(`decision:test-suite/crash-sweep-stays`).
