# Advanced mocking engineering

Engineering requirements for `epic:advanced-mocking`.

## What already holds

- The mock-data CLI writes the real hub and runner daemons' stores directly, through schema reflection, importing
  nothing from blizzard; every write is checked against the reflected schema first and lands in foreign-key order
  (`blizzard-mock/src/blizzard_mock/mock_data/internal/reflected_store.py`,
  `blizzard-mock/src/blizzard_mock/mock_data/domain/schema_contract.py`).
- The hub applies a configuration document idempotently (`blizzard hub config apply`), which the workspace's hub
  service already uses to declare the mock forge, its sources, and its repositories.
- The mock forge holds issues, pull requests, comments, labels, checks, and levers in memory, and has no bulk or
  create-by-key route ([`research/mock-composition/findings.md`](./research/mock-composition/findings.md)).

## Bulk apply (engineering:bulk-apply)

**Proposed.** An apply's cost grows with the tables and services it touches, not with the rows it writes. Rows reach a
store in batched statements inside one transaction per store, what an apply already holds is found with one read per
table rather than one per entity, the schema is checked once per table per apply, and the forge takes a batch of issues,
comments, and pull requests in one request. Counters a write must advance, such as a runner's outbox sequence, are
reserved for the whole batch at once. Serves `product:fixture-scale`, `product:fixture-reconcile`.

Acceptance:

- The number of statements an apply sends to a store does not grow with the number of rows it writes.
- A world of 100,000 chunks, each tied to a forge issue, applies to a sqlite store on a developer laptop in under ten
  minutes.
- Rerunning that apply on the same env, which writes nothing, finishes in under one minute.
