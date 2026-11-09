# Optimization engineering

The engineering requirements of `epic:test-suite/optimization`.

## What already holds

Paths here and in the spec are relative to the `blizzard` repo root unless they say otherwise.

The baseline, from successful pushes to `master` on 18 October 2026:

| Job                            | Duration                | Shape                                                                    |
| ------------------------------ | ----------------------- | ------------------------------------------------------------------------ |
| pytest unit + component        | 11.0–16.9 min per shard | 4 shards split by a hash of each test's id, `-n auto` on 4 vCPU each     |
| crash sweep, CI profile        | 7.0–7.2 min             | 43 tests, `-n auto` on 4 vCPU                                            |
| service tier                   | 3.2–5.8 min per shard   | 237 tests in 3 shards, `-n auto` on 4 vCPU, each test with its own stack |
| dev hub image                  | 3.0–4.5 min             | multi-arch QEMU build on every push to `master`                          |
| eslint + vitest + client drift | 1.9–2.2 min             | three `ng test` projects run one after another                           |

Every check starts at once, so the pull request's critical path is the slowest single job, a pytest unit + component
shard, and the crash sweep is next behind it. Queue time is about 2 s.

- `pr.yml` and `push.yml` each call `gate.yml` and `upper-tiers.yml` side by side, so every check starts at once, and a
  job in one workflow cannot wait on a job in the other.
- `gate.yml`'s `python-tests` job runs `uv run pytest -n auto` in four shards, `BLIZZARD_TEST_SHARD=<shard>/4`.
  `tests/support/suite_plugin.py`'s `pytest_collection_modifyitems` keeps a test in shard
  `zlib.crc32(nodeid) % count + 1` (`shard_of`), so the split is even by count and blind to time. `python-tests-result`
  reports the shards under `pytest (unit + component)`.
- Nothing records a test's duration: no `--durations`, no JUnit report, no artifact.
- The `frontend` job runs `npm run test`, which is `ng test hub && ng test runner && ng test fleet`, three projects in
  sequence. It builds the hub only inside `bundle-check` and Storybook inside `story-render`. The full `npm run build`
  first runs in `push.yml`'s `dev-build`, through `scripts/build-wheel.sh`.
- `push.yml`'s `dev-image` builds `linux/amd64,linux/arm64` under QEMU on every push.
- The hosted hub, the only consumer of `edge`, already runs on amd64: `blizzard-infra`'s
  `terraform/application/config/prod.tfvars`, a `t3a.small` on an amd64 AMI.
- `blizzard-context`'s `verification/blizzard/tier-rules.md` already asks the service tier for waits that end when the
  system speaks (`bzh:service-waits-on-the-system`).

## Brief requirements

- engineering:assertions-unchanged — No change in this epic alters what any test asserts. Serves `product:shorter-gate`.
- engineering:installations-untouched — Nothing an installation holds moves.
- engineering:blizzard-only — The epic changes nothing outside the `blizzard` repo, save the rule extension in
  `blizzard-context` (`engineering:waits-rule`).
- engineering:green-master — Every change leaves `master` green and every required check's name unchanged, save the UI
  check's rename (`engineering:shared-board-build`).
- engineering:reground-before-build — The spec and the baseline are taken again against the gate the tier epics leave
  behind before anything is built from them.

## The gate is measured (engineering:gate-measured)

The baseline above is the measure, taken again with the durations recorder after the split, the amd64-only image, and
the outliers land. Each of those changes records the job it shortened, before and after, in its description. Serves
`product:shorter-gate`.

Acceptance:

- Once every work area has landed, the gate's critical path is recorded against the baseline, whatever it came to.

## Shards balanced by duration (engineering:balanced-shards)

The unit and component shards, and the service tier's pytest half, are split by recorded durations rather than by a hash
of each test's id, which spreads tests evenly by count but not by time and leaves one shard six minutes longer than
another. The split changes no test. Serves `product:shorter-gate`.

Acceptance:

- Every collected test lands in exactly one shard.
- The same collection and table give the same partition in any item order.
- A missing or unreadable table falls back to the hash split with a warning.
- The largest shard's total is within the longest single test of the mean.
- A local `uv run pytest` with `BLIZZARD_TEST_SHARD` unset deselects nothing.

## An amd64-only dev image (engineering:amd64-dev-image)

The dev image built on every push to `master` is `linux/amd64` only, saving most of its QEMU time. Releases keep both
platforms, so the first arm64 build of any change is the release candidate's. Serves `product:arm64-releases`.

## The board built once per pull request (engineering:shared-board-build)

The `frontend` job builds both boards ahead of time on every pull request and hands the build to the UI job, which runs
on it rather than building its own. The UI job moves onto the shared build only once its runtime, recorded over its runs
in `upper-tiers.yml`, shows it finishing inside the slowest `python-tests` shard, so it never becomes the gate's
critical path (`decision:test-suite/optimization/ui-job-in-gate`). Serves `product:board-build-on-pr`.

## Waits that end when the thing has happened (engineering:event-driven-waits)

A fixed wait is either too long, and every run pays for it, or too short, and the test is flaky on a slow machine. The
three fixed waits outside the service tier that this epic owns, in `tests/unit/cli/test_cli.py` and
`tests/component/runner/harness/opencode/compatibility/test_diagnostic.py`, are each replaced by a wait on the event it
stood for, and a barrier event where an assertion is negative. E2E's `sleep(0.5)` tick spacing, its `poll_until`
intervals, and its 10 s termination allowance move with their scenarios under `epic:test-suite/fleet-tier`, and the
crash sweep's waits are untouched. Serves `product:shorter-gate`.

## The waits rule covers every tier (engineering:waits-rule)

`bzh:service-waits-on-the-system` is extended to every tier by name, in the change that lands the last of the three
waits: a test waits on an event, a state, or a process it can observe, and a bounded poll is its upper limit, never its
duration.

## Slow outliers, measured then decided per test (engineering:outlier-triage)

The outliers are named by a measurement on the CI shape, not by the figures recorded before it; the slowest fifty items
are recorded beside the baseline. For each outlier the question is whether the slow thing is the thing under test. When
it is, the test stays, and its cost is reduced only by doing less of the same thing, and only where the smaller input
still exceeds every buffer the assertion is about: a test that proves a process group is reaped must start a process
group, but one megabyte through each pipe may prove what six do. When it is not, the slow part is replaced by the fake
already in the file or by an in-memory double, and the test moves to the unit tier if nothing slow remains: a test whose
subject is how the adapter parses a session's output has no business spawning the session. The decision is recorded per
test in the commit message of the change that makes it, not in the file. Serves `product:shorter-gate`.

## Open questions

- Where the cost of `tests/component/runner/harness/opencode/compatibility/test_diagnostic.py` lies. It was recorded at
  about 164 s of CPU, one 35 s test and some thirty at 4–5 s, all spawning subprocesses, but the file as it stands holds
  twenty test functions, twenty-four items with one parametrized five ways, and its subprocess tests carry bounds of 5 s
  and fixed sleeps of at most 0.7 s. The cost is either in what the tests start (the pty fork, the Landlock probe, the
  six megabytes each pipe drains, the `CompatibilityDiagnostic` runs) or in a recent change that already cut it. The
  measurement answers it. — engineering
- Whether `bundle-check` can read its stats from the new build rather than building its own. The epic takes the saving
  only if `bundle-check.js` can be pointed at it without changing what it checks. — engineering
