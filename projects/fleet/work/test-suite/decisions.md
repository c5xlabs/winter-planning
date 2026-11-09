# Test suite decisions

## The order of the epics (decision:test-suite/epic-order)

The foundation lands first and the hub tier follows it. The runner tier comes after them because it needs real product
work, a host whose loop can be stepped from outside and reads of the state its tests check through the ORM, and the
fleet tier follows on that host. The journey follows the fleet tier. The controlled clock needs only what the hub and
runner tiers already built, the stepped sweeps and the stepped host, so it may start beside the fleet tier and the
journey rather than after them, and the journey may adopt it once both have landed. Optimization comes last: its work
speeds the tiers that stay in Python and the pipeline around them, but the gate it measures is the one the tier epics
leave behind, so its spec stays provisional until they land.

## The kill-9 crash sweep stays in Python (decision:test-suite/crash-sweep-stays)

The crash sweep asks whether a process killed at an exact line leaves its store consistent, and both halves of that
question belong to this implementation: its crash points mark boundaries in this code, and most of its invariants
describe how this code keeps its books. A rewrite would bring different boundaries and could keep every promise a person
sees while breaking every one of those invariants, so it would write a sweep of its own rather than inherit this one.
The promise a rewrite does have to keep, that a fleet killed in the night wakes with nothing lost and nothing done
twice, is the journey's, and the journey moves. Whatever the sweep borrows from a file the fleet tier deletes moves to a
Python support module first.

## Nothing is optimized on its way out (decision:test-suite/no-optimizing-the-doomed)

Work on code the initiative is about to delete is wasted; the fixed SSE waits, for one, are replaced as the tests that
hold them are ported, not before.

## The black-box line wins (decision:test-suite/black-box-wins)

Where speed and implementation-independence pull apart, independence wins, so a shortcut such as starting a hub from a
pre-migrated database never reaches the tier.

## The harness travels with the tests (decision:test-suite/harness-travels)

blizzard-context is what a fleet worker reads before writing a test, so it moves in step: the foundation redefines the
tier there, and every port re-points the citations of the tests it moves in the same change. The daemons a Playwright
run starts keep every invariant the pytest suite gives them today, the statement guards first among them.

## Public surfaces only (decision:test-suite/public-surfaces)

A test seeds and asserts only through HTTP, the CLI, and the mock fleet's endpoints. It never writes or reads the hub's
store, because a row the daemon did not write skips every cache, event, and derivation the real path runs, and the test
then proves a state the hub can never reach on its own. Test data is built by the test's own fixtures through those
surfaces, never by `blizzard-mock-data`, which serves agents and people testing by hand. Where a test needs state no
public surface can produce, the order of preference is the real flow, then a surface an operator or an integration could
use as well, shaped for the general case rather than for one test and secured like every other, and only when nothing
reasonable serves, a test-only route behind a configuration flag. Store access is allowed only in a `fresh-hub` test
whose subject is the store itself, against files before the daemon starts or after it stops.

## Each test moves once (decision:test-suite/move-once)

Porting a test and moving it onto the shared hub are one step: it lands in Playwright, at the loosest isolation level
its reads allow, and its Python original is deleted in the same change. Any helper another tier imports from the deleted
file moves to a support module first. The port keeps the original's function name as its title, so the harness citations
that name it move mechanically. A test whose subject is the whole hub moves to a hub of its own, `fresh-hub`, rather
than being weakened to fit a shared one.

## One package, its folders named for the kind of test (decision:test-suite/package-layout)

The Playwright package is rooted at `tests/`, and each folder beneath it names what a test holds real. A service test
brings up one real thing with mocks around it, in three shapes: the hub alone in `tests/service/hub/`, a runner host
alone in `tests/service/runner/`, and both together in `tests/service/fleet/`. A UI test, in `tests/ui/`, drives the
built board in a browser against a real hub, with the same mocks behind it. Nothing in the package is an end-to-end
test, because blizzard cannot yet stand up a real work source or a real GitHub repository under test, and until it can,
the word is not earned. The package shares `tests/` with the Python suites only until they have moved.

## The service tier stays one method with two runners (decision:test-suite/one-method)

`blizzard:service-test` keeps its name and its required CI check. It runs the Playwright package's service projects, and
beside it the pytest cases whose subject is Python itself: an adapter driven against a mock fleet binary, which
`bzh:external-cli-fake-is-service-tier` keeps in this tier because only its job checks out the mock fleet. From the day
the foundation lands, a new daemon test is written in Playwright.

## Playwright Test is the tier's framework (decision:test-suite/playwright)

Playwright Test's fixtures are the draw: scoped, injected by name, built only when a test asks for them, and composed
from one another, so a test that names a tenant gets the hub beneath it without arranging either. Its projects are the
natural shape for hub configuration profiles. Writing the tier in another language enforces the black-box line by
construction, since a TypeScript test cannot import `blizzard.*`. Playwright runs TypeScript without a build, and the
gate checks its types. The alternatives were weighed but not spiked:

- **pytest in a package sealed from `blizzard`** has the same fixture model and would keep most test bodies, but leaves
  the boundary to packaging rather than language.
- **Go**, with the standard library or Ginkgo, orchestrates processes and reads streams well but has no fixture model of
  comparable depth.
- **Rust**'s nextest runs each test in its own process, which fights a hub shared across tests.
- **Hurl and Venom** cannot hold an event stream open or own a daemon's lifecycle.

A spike against a real hub retired the hub side's risks and found one ceiling, a single hub's one core; the foundation
carries what it proved and what it asks of the product. The spike never reached the runner, so the runner tier opens
with one of its own.
