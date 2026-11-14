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

An apply's cost grows with the tables and services it touches and with its rows only in whole batches,
never one round trip per row, so a script building a hundred thousand chunks finishes in minutes. Serves
`product:fixture-scale`, `product:fixture-reconcile`. How the batches are made is in
[`spec/fixture-format.md`](./spec/fixture-format.md).

Acceptance:

- The round trips an apply makes to one table or one service grow at most as the number of its rows divided by a fixed
  batch size, plus a constant; a graph, posted alone, counts as one row of its own.
- A world of 100,000 chunks, each tied to a forge issue, applies to a sqlite store in under ten minutes on the reference
  machine.
- Rerunning that apply on the same env, which writes nothing, finishes in under one minute on the reference machine.
- A script applying a 100-chunk fixture 1,000 times with distinct prefixes reaches the same 100,000 chunks in under
  fifteen minutes on the reference machine, so what each apply costs before it writes a row is bounded too.

## Feature envs only (engineering:env-only-targets)

An apply writes only to a feature env's own stores and mocks, and contacts no other service.

Acceptance:

- An apply whose hub, forge, or stores are not the env's own, the hosted hub among them, is refused before anything is
  written, naming the target.
- No flag or variable turns the refusal off.

## The product's own write paths first (engineering:public-paths)

A record the hub can be told to create through its own API or configuration document is created that way,
so the hub enforces its own rules on it; the stores are written directly only for records with no such path.

Acceptance:

- Projects, sources, repositories, forges, secrets, urgencies, scopes, routines, cadences, graphs, and hub-authored work
  items reach the hub through its API or `config apply`.
- Each record an apply writes directly to a store is listed, with the reason it has no public path, in the spec.

## Open questions

- Which machine is the reference machine the bulk-apply times are measured on. The operator answers.
