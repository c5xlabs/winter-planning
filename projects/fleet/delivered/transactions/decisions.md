# Transactions decisions

## Tenant row as a mutex (decision:transactions/tenant-row-mutex)

The tenant row stays the mutex every write in a tenant locks until commit. Relaxing it so writers on different chunks
run side by side needs every writer reviewed case by case, so it is deferred out of this epic and taken up when a
deployment on Postgres needs the concurrency. The trade-off is in
[tenant row mutex](../../research/transactions/tenant-row-mutex.md).

## Cycle check lock scope (decision:transactions/cycle-check-lock)

Every change to chunk dependencies — a declare, a group, a delete, or a release — takes one store-held lock over all
tenant chunks, so no two dependency changes run at once. Other chunk writes keep locking only the chunks they name.

## Business rules leave the store first (decision:transactions/judgments-before-seam)

A few store methods decide a business rule themselves, such as refusing a write because the chunk is already stopped or
done, where the store should refuse a write only because its data changed underneath it and leave the rule to the
domain. Those rules move to the domain before the hub's writes move onto the one unit-of-work path. Each rule sits
inside a write that move relocates, so relocating the write first would carry the rule along with it, and the rule would
then have to be pulled out of its new home as well.

## A stop racing an urgency retirement (decision:transactions/urgency-move-race)

No stop or completion races an urgency retirement, and a terminal chunk keeps the urgency it ended with. The retirement
already locks every chunk it moves: it pre-reads the urgency's open holders, declares them, re-reads them under the
lock, and runs again over any it missed. A stop or completion takes the same chunk lock, so the terminal read the
retirement takes under it is exact, and moving that read from the store to the domain opens no window. An earlier form
of this decision accepted a race on the premise that the retirement locked only the urgency; the code never did.
