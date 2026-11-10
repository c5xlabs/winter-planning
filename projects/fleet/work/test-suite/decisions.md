# Test suite decisions

## The order of the epics (decision:test-suite/epic-order)

The foundation lands first, then the hub, runner, and fleet tiers and the journey in that order, with the controlled
clock free to start beside the fleet tier and optimization last, because each epic needs what the ones before it built
and optimization measures the gate the tier epics leave behind. Each epic's dependencies are in
[epic order](../../research/test-suite/epic-order.md).

## The kill-9 crash sweep stays in Python (decision:test-suite/crash-sweep-stays)

The kill-9 crash sweep stays in Python, because its crash points and most of its invariants describe this
implementation, so a rewrite would write a sweep of its own rather than inherit this one. The promise a rewrite does
have to keep, that a fleet killed in the night wakes with nothing lost and nothing done twice, is the journey's, and the
journey moves; the detail is in [tier rules](../../research/test-suite/tier-rules.md#the-crash-sweep).

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

A test seeds and asserts only through HTTP, the CLI, and the mock fleet's endpoints, never the hub's store outside a
`fresh-hub` test whose subject is the store itself, because a row the daemon did not write skips every cache, event, and
derivation the real path runs. Its data comes from its own fixtures through those surfaces, never from
`blizzard-mock-data`; the order of preference for a state no public surface produces is in
[tier rules](../../research/test-suite/tier-rules.md#public-surfaces).

## Each test moves once (decision:test-suite/move-once)

Porting a test and moving it onto the shared hub are one step: it lands in Playwright at the loosest isolation level its
reads allow and its Python original is deleted in the same change, while a test whose subject is the whole hub moves to
a hub of its own, `fresh-hub`, rather than being weakened to fit a shared one. How helpers and titles move is in
[tier rules](../../research/test-suite/tier-rules.md#moving-a-test).

## One package, its folders named for the kind of test (decision:test-suite/package-layout)

The Playwright package is rooted at `tests/`, each folder beneath it naming what a test holds real
(`tests/service/hub/`, `tests/service/runner/`, `tests/service/fleet/`, and `tests/ui/`), and nothing in it is called
end-to-end, because blizzard cannot yet stand up a real work source or a real GitHub repository under test. What each
folder holds is in [tier rules](../../research/test-suite/tier-rules.md#package-layout).

## The service tier stays one method with two runners (decision:test-suite/one-method)

`blizzard:service-test` keeps its name and its required CI check, running the Playwright package's service projects
beside the pytest cases whose subject is Python itself, an adapter driven against a mock fleet binary, which
`bzh:external-cli-fake-is-service-tier` keeps in this tier because only its job checks out the mock fleet. From the day
the foundation lands, a new daemon test is written in Playwright.

## Playwright Test is the tier's framework (decision:test-suite/playwright)

Playwright Test is the tier's framework, because its fixtures are scoped, injected by name, built on demand, and
composed, and a TypeScript tier enforces the black-box line by construction since it cannot import `blizzard.*`. The
alternatives weighed and the spike against a real hub, which reached only the hub side, are in
[framework](../../research/test-suite/framework.md).
