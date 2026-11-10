---
name: durations-recorder
code:
  - blizzard/.github/workflows/gate.yml::python-tests
  - blizzard/.github/workflows/upper-tiers.yml::service-tier
---

# The durations recorder

Records every test's duration on every gate run, changing no shard's membership. Builds `spec/shards.md` §The recorder,
the recorder's half of §CI, and the per-run file in §Data, toward `engineering:balanced-shards` and
`engineering:gate-measured`.

## Change

- `tests/support/suite_plugin.py` gains `BLIZZARD_DURATIONS_OUT` and the recorder's `pytest_runtest_logreport` and
  `pytest_sessionfinish`, merging the xdist workers' parts.
- `python-tests` and `service-tier` set `BLIZZARD_DURATIONS_OUT=durations.json` and upload `durations-<job>-<shard>` on
  every run that is not cancelled, kept 14 days.
- No shard's membership changes.

## Acceptance

- A run with `BLIZZARD_DURATIONS_OUT` set writes each executed test exactly once, and no skipped test.
- A run with it unset writes nothing.
- A red run's artifact is uploaded.
