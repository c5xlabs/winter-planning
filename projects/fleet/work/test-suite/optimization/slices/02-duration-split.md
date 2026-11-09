---
name: duration-split
depends-on:
  - durations-recorder
---

# The committed table and the split

Splits the shards by recorded duration, once ten `push` runs on `master` carry the recorder's artifacts. Delivers
`engineering:balanced-shards`; builds `spec/pipeline.md` §The committed table and §The split.

## Change

- `tests/durations.json`, built by `scripts/test-durations.py refresh` and `mise run test-durations`.
- The weekly `.github/workflows/durations.yml` refresh.
- `pytest_collection_modifyitems` assigns shards by longest-processing-time packing, falling back to `shard_of`.
- `tests/guards/test_shard_split.py`.
- The change's description records each shard's time before and after, from the runs either side of it
  (`engineering:gate-measured`).

## Acceptance

- `tests/guards/test_shard_split.py` passes over a synthetic collection: every item in exactly one shard, the same
  partition from any item order, a missing table falling back to the hash, and the largest shard's total within the
  longest single test of the mean.
