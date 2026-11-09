---
name: durations-recorder
---

# The durations recorder

Records every test's duration on every gate run, changing no shard's membership. Builds `spec/pipeline.md` §Recording
durations, toward `engineering:balanced-shards` and `engineering:gate-measured`.

## Change

- The suite plugin's recorder, written when `BLIZZARD_DURATIONS_OUT` names a file, merged across xdist workers.
- `python-tests` and `service-tier` upload `durations-<job>-<shard>` on every run that is not cancelled, kept fourteen
  days.
- No shard's membership changes.

## Acceptance

- A run with `BLIZZARD_DURATIONS_OUT` set writes each executed test exactly once, and no skipped test.
- A red run's artifact is uploaded.
