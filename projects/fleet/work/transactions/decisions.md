# Transactions decisions

## Tenant row as a mutex (decision:transactions/tenant-row-mutex)

**Open.** Whether the tenant row stays the mutex every write in a tenant locks until commit, or is relaxed so writers on
different chunks run side by side; the epic proceeds with it kept, and the call is taken when a deployment on Postgres
needs the concurrency. The trade-off is in [tenant row mutex](../../research/transactions/tenant-row-mutex.md).

## Cycle check lock scope (decision:transactions/cycle-check-lock)

**Open.** Whether the cycle check's store lock is one keyed lock per tenant, which serializes every dependency change in
the tenant as the in-process lock does today and needs no revision, or a narrower lock, which needs a way to name the
part of the graph a change can close a cycle through; it is decided when the exclusions move and blocks
`engineering:store-held-exclusions`.

## Store judgments move first (decision:transactions/judgments-before-seam)

The store's judgments are handed back to the domain before the seam converges, because each sits inside a write the
convergence later moves, and moving a write that still decides a rule moves the rule with it.
