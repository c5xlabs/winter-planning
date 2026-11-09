# Transactions engineering

Engineering requirements for `epic:transactions`.

## What already holds

Measured on `master` at `7d984dad8`:

- 25 repository port methods exist only as a `*_locked` twin of a write.
- 17 domain and delivery modules reach the store through the legacy locked seam (`IChunkExclusiveWrites`); 12 open the
  unit of work directly, and several do both.
- Four store methods judge a domain predicate rather than refusing only a stale write.
- Two exclusions live in the hub process's memory: the per-tenant lock that keeps the dependency graph free of cycles,
  and the lock that keeps an operator's reset of the export cursor from being overwritten by a pass in flight.
  `bzh:store-exclusive-write` lists them as residual debt.
- One hub process serves the store, and every test runs on SQLite, whose single writer lock serializes every write.
- `bzh:write-reestablishes-read` prefers compare-and-set to a lock where one version or monotonic token covers a
  decision.
- `blizzard:postgres-uow-concurrency` is the focused Postgres proof of the unit of work, and the gap ledger (`gaps.md`)
  keeps it a focused method rather than a Postgres test tier.
- A lock that needs a store row no table holds lands as an expansion (`bzh:store-expand-contract`); a row in
  `keyed_locks` needs no revision.

## One write entry (engineering:one-write-entry)

`IHubTransactions.begin` opens every multi-repository change, and its single lock declaration is the only way a unit
takes a lock. Every repository write joins a caller's unit when given one and opens its own short write otherwise; a
write correct only under a lock requires a locked unit. The legacy locked seam and every `*_locked` twin are removed,
each twin folded into the plain method it shadowed, and the guard reads a locked handle offers become ordinary
repository reads that join the unit. Serves `product:one-way-to-write`.

Acceptance:

- No `*_locked` method and no locked-handle type remains under `hub/`.
- Every write that must win exactly once takes its locks through one `begin` declaration.

## Store refuses only stale writes (engineering:store-refuses-stale)

A store write is refused only because it is stale; the domain alone decides whether it is allowed. The four store
methods that judge a domain predicate hand their fact back to the domain, and the store conditions its write on the
token the domain read. A decision covered by one version or monotonic token takes the compare-and-set form before the
locked form. Serves `product:one-way-to-write`.

The four judgments:

- The terminal check inside `fence`.
- The escalation that runs that fence on its own write.
- The backlog write that drops a promoted chunk's position.
- The promote that writes nothing for an already-promoted chunk.

## Announce after commit (engineering:announce-after-commit)

An event is published only after the write it describes has committed, by the code that asked for the write. A unit
returns what it changed, and its caller publishes from that. The store's own path to the broker, the publisher a scoped
write carries and the callback a store queues on its commit, is removed, and the event it announces today moves to its
caller. Serves `product:one-way-to-write`.

## Adapters stay apart (engineering:adapters-stay-apart)

No adapter writes another repository's facts through a sibling's private helper. Each such write is handed back to the
domain, which composes both repositories in one unit; `WorkItemStore`'s composite mints are the largest case. Serves
`product:one-way-to-write`.

## Exclusions live in the store (engineering:store-held-exclusions)

Every exclusion a store decision depends on is a lock the store holds, so a second process would meet it. The dependency
graph's cycle check, the delete and release that share its lock, and the export cursor's pass lock move onto store
locks, and the in-process locks `bzh:store-exclusive-write` lists as residual debt are removed. The sweep locks, which
order a process's own bookkeeping and guard no store decision, stay in process. Where the cycle check's lock is scoped
is `decision:transactions/cycle-check-lock`. Serves `epic:transactions`.

## Locking shown on Postgres (engineering:postgres-locking-proof)

`blizzard:postgres-uow-concurrency` covers what SQLite cannot show: the closed-tenant check a scoped write opens with,
the ordered locks that keep two closes from closing the last two open tenants, the drain's delete order under enforced
foreign keys, and keyed locks of different keys not excluding each other. A gap the proof closes is removed from
`gaps.md`, and a mechanism the proof finds broken is fixed in this epic. Serves `epic:transactions`.

## Rules in final form (engineering:final-form-rules)

`bzh:transaction-participation`, `bzh:store-exclusive-write`, and `bzh:store-refuses-stale` lose their legacy clauses
and their Known debt, and each departure they catch at review gains a mechanical gate. Serves
`product:one-way-to-write`.

Acceptance:

- A gate fails on a `*_locked` name or a locked-handle type under `hub/`.
- A gate fails on a store write that reads a status-family fact to refuse.
- A gate fails on a broker call inside a store write.

## Behavior preserved (engineering:behavior-preserving-landings)

Every landing preserves behavior, and the old write path is removed within this epic. Serves
`product:no-visible-change`.
