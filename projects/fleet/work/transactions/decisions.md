# Transactions decisions

## Tenant row as a mutex (decision:transactions/tenant-row-mutex)

**Open.** Whether the tenant row stays a mutex. Every write in a tenant opens by locking that tenant's row and holds it
to commit, so on Postgres every writer in one tenant queues behind the one before it and the chunk and keyed locks
beneath never contend. Keeping it is simple and safe and makes those finer locks a second line of defence; relaxing it
lets writers on different chunks run side by side, but costs an existence read on every write to keep refusing a closed
tenant and makes every finer lock load-bearing for the first time. This epic proceeds with the row kept as a mutex, and
the call is taken when a deployment on Postgres needs the concurrency.

## Cycle check lock scope (decision:transactions/cycle-check-lock)

**Open.** Where the cycle check's store lock lives. One keyed lock per tenant serializes every dependency change in that
tenant, as the in-process lock does today, and needs no revision; a narrower lock needs a way to name the part of the
graph a change can close a cycle through. It is decided when the exclusions move, and it blocks
`engineering:store-held-exclusions`.

## Store judgments move first (decision:transactions/judgments-before-seam)

The store's judgments are handed back to the domain before the seam converges, because each sits inside a write the
convergence later moves, and moving a write that still decides a rule moves the rule with it.
