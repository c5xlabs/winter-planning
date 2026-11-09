---
epic: transactions
refinement: outline
status: planned
---

# Transactions

A standalone epic; its requirements are inline below.

## The problem

An agent adding a write to the hub (`persona:fleet-agent`) has to learn which of several ways to change the store this
write should take, and the code offers it every one of them as a precedent. It can open a unit of work and declare its
locks, which is the path the harness asks for. It can open a locked handle and call one of the locked twins beside the
method it actually wanted, which is the path most of the existing decisions still take. It can call a repository method
that opens its own short write. It can find a store method that decides the business rule itself, checking that a chunk
is terminal inside its own query, and copy that. And it can find an event announced from deep inside a store write,
where every other event is announced by the caller once the write has committed. Each path is correct where it stands.
Together they teach a newcomer that any of them will do, and the rules that say otherwise are written as a list of
exceptions a reviewer has to remember.

Everything holds today only because one hub process serves the store and every test runs on SQLite, whose single writer
lock serializes everything whether or not the code asked it to. Two exclusions live in the hub process's memory rather
than in the store, where a second process would never meet them.

## The outcome

Blizzard reaches the final state of how it changes its own data. A write opens one way. A decision that must win exactly
once locks what it reads, in one declared order, through the one unit of work. The store refuses a write only because it
is stale, and the domain alone decides whether it is allowed. An event is announced only after the write it describes
has committed, by the code that asked for the write. Every exclusion a writer depends on lives in the store rather than
in a process's memory. And the locking that SQLite makes true by construction is shown to hold on Postgres.

## What this epic is not

Running a second hub process. This epic makes the store the only arbiter of a write, but the tenant cache, the event
broker, the background sweeps, and the boot recovery of open turns stay single-process by construction. Nor does it
touch the runner's store, which one process serves by design, or crash correctness, which blizzard's crash-correctness
rules own separately from any lock.

## Requirements

### One way to write (product:one-way-to-write)

An agent adding a write to the hub finds one way to open it and one way to announce what it changed, and the rules it
reads state that way without a list of exceptions.

Acceptance:

- The hub's code offers one precedent for opening a write and one for announcing it.
- The rules for writing to the store carry no legacy clauses or known debt.

### Nothing visible changes (product:no-visible-change)

The write path is not a released surface: no route, command, wire model, or configuration changes, and every landing
preserves behavior, so removing the old path is this epic's own last step rather than a follow-up break.

Acceptance:

- Every route, command, wire model, and configuration a supported client uses behaves as before each landing.
