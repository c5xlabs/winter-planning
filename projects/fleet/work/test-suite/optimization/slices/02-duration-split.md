---
name: duration-split
depends-on:
  - durations-recorder
code:
  - blizzard/tests/support/suite_plugin.py::pytest_collection_modifyitems
  - blizzard/tests/support/suite_plugin.py::pytest_ignore_collect
  - blizzard/tests/support/suite_plugin.py::shard_of
---

# The committed table and the split

Splits the shards by recorded duration, once ten `push` runs on `master` carry the recorder's artifacts. Delivers
`engineering:balanced-shards`; builds `spec/shards.md` §The split, §The refresh script, `durations.yml` in §CI, and the
committed table in §Data. Shaped by `decision:test-suite/optimization/committed-durations-table`; waits on the open
question in `spec/shards.md` of whether files or tests are packed.

## Change

- `tests/durations.json`, built by `scripts/test-durations.py refresh` and `mise run test-durations`.
- The weekly `.github/workflows/durations.yml` refresh.
- `pytest_collection_modifyitems` assigns shards by `partition`'s longest-processing-time packing, falling back to
  `shard_of` with a warning when the table is missing or unreadable.
- `tests/guards/test_shard_split.py`.
- The change's description records each shard's time before and after, from the runs either side of it
  (`engineering:gate-measured`).

## Acceptance

- `tests/guards/test_shard_split.py` passes over a synthetic collection: every item in exactly one shard, the same
  partition from any item order, a missing table falling back to the hash, and the largest shard's total within the
  longest single test of the mean.
- A local `uv run pytest` with `BLIZZARD_TEST_SHARD` unset deselects nothing.
