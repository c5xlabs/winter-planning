# Converging each area

What each convergence slice converts, in the form [write-entry.md](./write-entry.md) defines. Paths are under
`blizzard/src/blizzard/hub/`, measured on `master` at `eda353d55`: 23 `*_locked` port methods across 11 store classes,
and 27 legacy lock sites in 18 domain and delivery modules.

A twin folds into its plain method where one exists, and otherwise is renamed to drop `_locked`. Its handle parameter
becomes `transaction: LockedTransaction`, except where noted. A plain method that only tests call is folded into, not
kept beside.

## Chunk lifecycle (`converge-chunk-lifecycle`)

| Twin                       | Becomes                                                                       | Store                         |
| -------------------------- | ----------------------------------------------------------------------------- | ----------------------------- |
| `record_pause_locked`      | `record_pause(chunk_id, *, paused, by, at, transaction: Transaction \| None = None)` — resume keeps calling it alone | `chunk_lifecycle_store.py:41` |
| `record_stop_locked`       | `record_stop`                                                                 | `chunk_lifecycle_store.py:50` |
| `record_completion_locked` | `record_completion`                                                           | `chunk_lifecycle_store.py:69` |

Lock sites that switch here: `PauseService.pause` (`domain/operations/pause.py:53`), `StopService.stop` (`stop.py:54`),
`CompleteService.complete` (`complete.py:54`).

## Chunk movement (`converge-chunk-movement`)

| Twin                       | Becomes                                                                            | Store                          |
| -------------------------- | ---------------------------------------------------------------------------------- | ------------------------------ |
| `record_transition_locked` | `record_transition` — the plain method has no `src` caller; its tests move         | `chunk_movement_store.py:107`  |
| `record_migration_locked`  | `record_migration` — likewise test-only today                                      | `chunk_movement_store.py:248`  |
| `record_restart_locked`    | `record_restart`; `record_restart_conn` stays private to the store                 | `chunk_movement_store.py:379`  |
| `record_requeue_locked`    | `record_requeue`                                                                   | `chunk_movement_store.py:470`  |

No lock site switches here: every caller also calls a twin of another area (§Cross-area sites).

## Chunk route (`converge-chunk-route`)

| Twin                            | Becomes                                                          | Store                       |
| ------------------------------- | ---------------------------------------------------------------- | --------------------------- |
| `record_route_locked`           | `record_route`                                                   | `chunk_route_store.py:119`  |
| `record_route_released_locked`  | `record_route_released`                                          | `chunk_route_store.py:155`  |
| `record_route_token_locked`     | `record_route_token`                                             | `chunk_route_store.py:185`  |

`IReadRunnerRegistry.get_runner` gains `transaction`. Lock sites that switch here: `ClaimService.claim` with
`_claim_locked` (`domain/execution/claim.py:383`), `ClaimService.rekey` (`claim.py:478`), `ReleaseService.release`
(`release.py:79`), `DetachService.detach` and `release_held` (`detach.py:70,85`). `ClaimService.exclusive` is removed.

## Chunk dependencies (`converge-chunk-dependencies`)

| Twin                 | Becomes                                                                               | Store                             |
| -------------------- | ------------------------------------------------------------------------------------- | --------------------------------- |
| `record_fold_locked` | `record_fold(..., transaction: LockedTransaction)`; it keeps checking that `grouped_into` and every target are locked | `chunk_dependencies_store.py:164` |

`list_standing_edges` and `is_ephemeral` gain `transaction`. Lock site that switches here: `DependencyService.declare`
(`domain/chunk/dependencies.py:155`), which calls no twin and reads only through the handle.

## Decisions and escalations (`converge-decisions-and-escalations`)

| Twin                              | Becomes                                                                     | Store                              |
| --------------------------------- | --------------------------------------------------------------------------- | ---------------------------------- |
| `record_decision_locked`          | `record_decision` — test-only today                                         | `chunk_decisions_store.py:257`     |
| `record_bounce_locked`            | `record_bounce`                                                             | `chunk_escalations_store.py:129`   |
| `record_bounce_escalation_locked` | `record_bounce_escalation`                                                  | `chunk_escalations_store.py:193`   |

`open_graph_gate_on_arrival` (`domain/execution/gate_arrival.py:13`) takes `transaction: LockedTransaction` in place of
the handle. Lock site that switches here: `DecisionService.submit` (`domain/execution/decisions.py:186`).

## Hub execution (`converge-hub-execution`)

| Twin                                | Becomes                                                       | Store                           |
| ----------------------------------- | ------------------------------------------------------------- | ------------------------------- |
| `record_hub_step_transition_locked` | `record_hub_step_transition`                                  | `chunk_hub_exec_store.py:47`    |
| `record_hub_node_poll_locked`       | `record_hub_node_poll`                                        | `chunk_hub_exec_store.py:152`   |
| `IWriteFlows.end_turn_locked`       | `end_turn`; it resolves the turn's chunk and checks that chunk is locked, as today | `flow_store.py:441` |

`ITurnEnd.end` (`delivery/hub_node.py:242`) and `_TurnEnding` (`delivery/flow_turn.py:130`) take the transaction in
place of the handle. `FlowStore.take_turn` stops locking `flow` for itself; its caller declares the key. Lock sites that
switch here: `TurnRunner._end_unrouted` and `_end` (`flow_turn.py:686,693`), `FlowLineReconciler.end_unanswered`
(`domain/delivery/reconcile.py:223`).

## Record and work refs (`converge-record-and-work-refs`)

| Twin                           | Becomes                                                                        | Store                           |
| ------------------------------ | ------------------------------------------------------------------------------ | ------------------------------- |
| `mint_locked`                  | `mint(chunk, *, transaction: LockedTransaction)`, declared on the port; it keeps checking every pointer key is locked | `chunk_record_store.py:316` |
| `set_graph_locked`             | `set_graph`                                                                    | `chunk_record_store.py:321`     |
| `set_defaults_locked`          | `set_defaults`                                                                 | `chunk_record_store.py:329`     |
| `set_intended_migration_locked`| `set_intended_migration`                                                       | `chunk_record_store.py:368`     |
| `add_work_refs_locked`         | `add_work_refs`                                                                | `chunk_work_refs_store.py:150`  |

`live_holders` and `projects_linking_sources` gain `transaction`; the `work_ref` and `project-source-links` key builders
move to the domain. Lock sites that switch here: `IngestService.ingest` (`domain/chunk/ingest.py:278`, the one
`locked_work_refs` caller) and `EditService.edit` (`domain/operations/edit.py:297`).

## The graph, and the seam's removal (`converge-graph`)

| Twin                               | Becomes                                                                    | Store                  |
| ---------------------------------- | -------------------------------------------------------------------------- | ---------------------- |
| `IWriteGraphRepository.mint_locked`| `mint(graph, *, definition_yaml, at, transaction: LockedTransaction)`      | `graph_store.py:294`   |

`get_enabled_by_name` and `any_minted` gain `transaction`; `GraphStore` stops opening its own write for `locked_name`.
Lock site that switches here: the default-graph mint in `domain/graph/authoring.py:206`.

### Cross-area sites

Each calls twins of more than one area, so it passes `handle.transaction` through the area slices and switches here:

| Site                                                    | Twins it calls                     |
| ------------------------------------------------------- | ---------------------------------- |
| `ApplyService._transition` (`domain/execution/apply.py:398`) | transition, decision (gate)    |
| `ApplyService._land_migration` (`apply.py:580`)         | migration, decision (gate)         |
| `RestartService.restart` (`domain/operations/restart.py:210`) | restart, decision (gate)     |
| `RequeueService.requeue` (`domain/execution/decisions.py:296`) | requeue, route released     |
| `GroupService.group` with `_group_locked` (`domain/operations/queue.py:799`) | work refs, fold |
| `HubNodeRouter` — `_route_away`, `_commit`, `refuse`, `record_wait`, `_escalated`, `_unroutable` (`delivery/hub_node.py:399,427,622,746,871,947`) | migration, decision (gate), bounce, bounce escalation, hub step, poll, end turn |

The `ILockedChunkRead`-typed helpers in `hub_node.py` (`_escalate_locked`, `_bounce`, `_commit_on`, `_migrate_on`,
`_route_on`, `_standing_visit`, `_escalated_on`, `_unroutable_on`) take the transaction with them. Then the seam goes:
everything [write-entry.md](./write-entry.md) §Removed lists.

## Tests

About 56 test files reach the seam; the heaviest are
`tests/component/hub/store/internal/test_flow_store_turn_writes.py`, `test_chunk_dependencies_store.py`,
`tests/support/store_read_census.py`, and `tests/support/load_all_facts_seed.py`. Some open
`stores.exclusive.locked(...)` directly (`tests/component/hub/test_command_node.py:178`, `test_rescoped_keys.py:246`).
Each slice moves the tests of what it converts; a test of a folded plain method is rewritten against the surviving one.

## Open questions

- **Where the cross-area sites switch.** Eleven of the 27 sites land in `converge-graph`, which then carries the
  hub-node router and most of the movement callers. The alternative is a slice of their own, after the area slices and
  before the graph.
- **Config apply's key lock.** `config_apply_store.py:145` takes its key set on a write it opens itself, so it departs
  from the one declaration, and no slice converges it.
