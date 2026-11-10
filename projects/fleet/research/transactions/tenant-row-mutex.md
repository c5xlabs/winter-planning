# Tenant row mutex

The trade-off behind `decision:transactions/tenant-row-mutex`.

Every write in a tenant opens by locking that tenant's row and holds it to commit. On Postgres every writer in one
tenant therefore queues behind the one before it, and the chunk and keyed locks beneath never contend.

- **Keeping the mutex** is simple and safe, and makes those finer locks a second line of defence.
- **Relaxing it** lets writers on different chunks run side by side, but costs an existence read on every write to keep
  refusing a closed tenant, and makes every finer lock load-bearing for the first time.
