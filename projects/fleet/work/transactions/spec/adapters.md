# Adapters stay apart

Which store adapters write another repository's facts, and how each write moves to the domain, for
`adapters-stay-apart`. Delivers `engineering:adapters-stay-apart`. Paths are under `blizzard/src/blizzard/hub/`,
measured on `master` at `eda353d55`.

An adapter writes only its own repository's tables. A write that spans repositories is composed by the domain, which
opens one unit through `IHubTransactions.begin` and calls each repository with it (`bzh:unit-of-work`). Reading a
sibling's rows, or embedding a sibling's lookup as a subquery of one's own insert, is not a cross-adapter write.

## Domain

### `WorkItemStore`'s composite mints

| Method                                       | Writes beyond `work_items`                                                                  | Caller                                                                 |
| -------------------------------------------- | ------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| `create_with_chunk` (`work_item_store.py:104`) | `chunks`, `chunk_work_refs` (`chunk_rows.insert_chunk_rows`)                              | `WorkItemEditService.create` (`domain/work_items/editing.py:196`)      |
| `create_run_with_chunk` (`:146`)             | the above, and `work_item_runs` (`run_context_store.insert_run_context_row`)                | `RunService.run` (`domain/garden/runs/run.py:267`); `CadenceMint.mint` (`domain/cadence/mint.py:203`) |
| `accept_create` (`:287`)                     | `garden_proposal_closures` and a lock on `garden_proposals` (`insert_garden_proposal_closure_row`), then the above | `WorkItemEditService.accept_create` (`editing.py:208`), from `GardenProposalClosureService.accept` (`domain/garden/proposals/closure.py:320`) |

Each becomes a domain composition in one unit over the work-item, chunk-record, run-context, and proposal-closure
repositories (`IWriteWorkItemRepository`, `IWriteChunkRecordRepository`, `IWriteRunContextRepository`,
`IWriteGardenProposalClosureRepository`), each write taking the unit's transaction. The work-item port loses its three
composite methods for a plain create; the proposal-closure port gains the accept-with-mint closure `accept_create`
writes today. `accept_create`'s insert-if-absent closure stays the gate on the rest: when it finds the proposal already
closed, the unit writes nothing further and the domain raises `GardenProposalAlreadyClosed`, as today.

### Other adapters writing a sibling's tables

| Adapter                                                  | Sibling's tables it writes                                    | Through                                          |
| -------------------------------------------------------- | ------------------------------------------------------------- | ------------------------------------------------ |
| `RoutineScopeStore.link`, `unlink` (`routine_scope_store.py:97,104`) | `routine_scopes`; CAS on `routines.revision`      | `routine_store.link_scope`, `move_revision`      |
| `RoutineStore.create`, `update` (`routine_store.py:222,261`) | `scopes`, `config_changes`                                | `scope_store.ensure_scope`                       |
| `ReviewFindingsStore.deliver` (`review_findings_store.py:80`) | `scopes`, `config_changes`; `findings`, `finding_facts` directly | `ensure_scope`; inline inserts          |
| `GardenDeliveryStore.deliver` (`garden_delivery_store.py:75`) | `findings`, `finding_facts`, `finding_sets`, `garden_proposals`, `garden_proposal_findings` directly; a lock on `findings` | `finding_store.lock_findings`; inline inserts |
| `TenantAdministrationStore` (`tenant_admin_store.py:37`)  | graph, project, urgency, and tenant-settings rows; `work_item_sequence` directly | adapters built on its connection (`_TenantCreation`, `:73`); `seed_hub_sequence` (`:102`) |
| `ConfigApplyStore` (`config_apply_store.py:43`)          | the cadence, forge, project-link, project, repository, routine, scope, settings, urgency, and work-source tables | those adapters' write bodies |
| `ConfigImportStore` (`config_import_store.py:34`)        | forge, project-link, repository, work-source, and secret tables | `insert_forge`, `insert_link`, `insert_record`, `insert_secret` |

### Shared row helpers

`chunk_rows` and `flow_rows` are shared modules, not adapters, and several chunk adapters write through them:
`record_hub_lease` (`lease_facts`, `epoch_owners`), `enqueue_close_intents` (`work_item_closures`, `close_intents`,
`event_log`), `withdraw_waiting` and `close_submission` (flow closure rows). They stay shared: each writes facts one
chunk write owes atomically, under the chunk lock its caller holds. `config_change_store.append_change`, the audit row
twelve adapters append beside their own write, stays shared for the same reason.

## Open questions

- **How far the slice reaches.** The engineering requirement names `WorkItemStore` as the largest case. The table above
  holds seven more; config apply and import are a bulk transaction by design and may stay a declared exception.
