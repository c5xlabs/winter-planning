---
name: report-builder
depends-on:
  - hub-sessions-routes
  - traced-selection
code:
  - blizzard/src/blizzard/runner/leases/session.py::IReadLeaseSessionRepository.pool_head
---

# Report builder

Adds the one builder that assembles the `schema: 1` report from a configuration, its bindings, harness health, the hub
sessions read, and the pool heads ([command.md](../spec/command.md) §JSON form, §Rows;
[resolution.md](../spec/resolution.md) §Pools already running). Delivers `product:tier-table`, `product:effort-table`,
`product:session-pools`, `product:cell-provenance`, `product:answer-without-hub`, and `product:chunk-view`. Shaped by
`engineering:read-only-report`. Waits on the open question of whether a live pool keeps its model or rotates on drift.

## Change

- The builder produces the harness, tier, effort, and pool rows and the `notes` findings.
- With a chunk named, it reads each session's pool head through `pool_head` and marks where it differs from the next
  fresh mint.
- An unreachable or unsupported hub sets `hub.status` and drops only the pools.

## Acceptance

- A tier a harness cannot resolve appears as `unmapped` with an `unmapped-tier` note.
- An effort a harness drops appears as `dropped` with a `dropped-effort` note.
- Every harness in a session's set is listed with what it would run; the selected one is marked, with each earlier
  member's skip reason.
- A chunk default appears as the source of any field it supplies.
- No resolved value appears without a source.
- With the hub unreachable, the harness, tier, and effort rows are complete.
- With a chunk named, a live pool's recorded model appears beside the next fresh mint's.
- The builder opens no write path to the runner store.
