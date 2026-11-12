# Transactions spec

What each slice builds, measured against `blizzard` on `master` at `eda353d55`.

| File                                       | Covers                                                                              | Slices                                |
| ------------------------------------------ | ----------------------------------------------------------------------------------- | ------------------------------------- |
| [store-judgments.md](./store-judgments.md) | Every store-side judgment handed back to the domain, by family                      | `store-judgments`, `config-judgments`, `garden-and-work-item-judgments`, `flow-judgments`, `tenant-judgments` |
| [write-entry.md](./write-entry.md)         | The target write form, the transitional form, and everything the legacy seam folds into | `converge-*`                      |
| [areas.md](./areas.md)                     | Each area's `*_locked` twins, lock sites, and reads, and the cross-area sites       | `converge-*`                          |
| [adapters.md](./adapters.md)               | The adapters that write a sibling's tables, and the domain compositions replacing them | `adapters-stay-apart`              |
| [exclusions.md](./exclusions.md)           | The dependency lock and the export cursor                                           | `store-held-exclusions`               |
| [hub-locks.md](./hub-locks.md)             | The hub lock table, the identity and tenant-lifecycle locks, and their lock order   | `tenant-judgments`                    |
| [postgres-proof.md](./postgres-proof.md)   | The four cases the Postgres proof grows to cover                                    | `postgres-proof`                      |
| [gates.md](./gates.md)                     | The two mechanical gates                                                            | `final-form-rules`                    |

The announce-after-commit change has no file: `5ae6ebf28` already removed the store's path to the broker, and
`announce-after-commit` has nothing left to build beyond confirming its acceptance.
