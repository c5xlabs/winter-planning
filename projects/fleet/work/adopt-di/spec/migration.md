# Mixed ownership and retirement

## System

```text
canonical provider (legacy leaf factory OR area factory, never both)
       ↓
typed service/port resolution ──────────────→ narrowed consumer
       ↓
legacy bag assembler ──────────────────────→ consumer not yet narrowed

FORBIDDEN: adopted provider → legacy bag → adopted provider
```

| Transitional object  | Responsibility                                                                                                                    |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Legacy leaf provider | Calls an extracted existing constructor/factory; takes concrete dependencies, never obtains them by constructing an aggregate bag |
| Adopted provider     | Owns the same key in its area's wiring; replaces, rather than overrides alongside, the legacy provider                            |
| Legacy bag assembler | Obtains canonical values; retains only fields with live consumers; may depend on adopted providers but is never their source      |
| Process root         | Calls the legacy area registration while needed, alongside adopted areas                                                          |
| Inventory gate       | Records registration owners and bag consumers; refuses duplicate ownership and uncategorized remaining legacy use                 |

An exported legacy dependency is factored into a leaf provider before an adopted provider can depend on it. A leaf's
existing constructor is called once through its canonical provider; its old construction branch is removed in the same
change. Aggregate factories may continue constructing unexported legacy internals, but no provider reaches into those
aggregates to obtain an adopted service's dependency.

## Domain

```text
Representative mixed tenant graph:

borrowed process clock        tenant ScopedConnections       tenant transactions
          │                           │                             │
          │                    FindingStore provider                 │
          │                     ├─ read alias                        │
          │                     └─ write alias                       │
          │                           │                             │
          └──────────────────── FindingExitService ─────────────────┘
                                      │
                ┌─────────────────────┴────────────────────┐
          findings route                      legacy TenantStores/TenantServices
                                                   │
                                          remaining old consumer
```

The store provider owns construction; its read and write aliases share identity. The exit service has no dependency on
either bag. Its old constructor call is removed; any retained bag field is filled from the canonical service. The same
shape applies when a legacy leaf, rather than an adopted store, supplies an unmoved collaborator.

| Object class                                                                                                                  | End-state treatment                                                                                                                                                     |
| ----------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `TenantServices`, `HubServices`, `TenantStores`, `TenantServiceParts`, `ServedTenant`, `TenantSweeps`, `TenantServicesOpener` | Retire composition/service-locator bags and generic tenant-service access                                                                                               |
| `HostProcess`, `RunnerProcess`                                                                                                | Separate runtime ownership from collaborator aggregation; narrow app/driver inputs; retain a lifecycle owner and runner roster, not service bags forwarded to consumers |
| `RunnerStores`, `HostStores`, read-store bundles                                                                              | Replace broad consumer dependencies with named repository ports; retain adapter-internal aggregation only where it implements a bounded store capability                |
| `LoopContext`, per-step Protocols, operation configuration and harness-set snapshots                                          | Retain typed driver state; construct collaborators through registered factories; do not turn the context into a container lookup surface                                |
| `ChunkReadStores` and API helper/auth records                                                                                 | Narrow callers to the required read seams; request auth records carry identity/scope, not a services bag                                                                |
| Plain records, results, settings, ownership handles, acquired transactions                                                    | Ordinary values; not all object construction is collaborator construction                                                                                               |

### Area ownership

The current source baseline is blizzard `4645daa28`. Rows identify finite ownership boundaries, not a landing sequence.
Each includes its registration factories, read aliases, and helper consumers; a provider used by several rows has one
owner and is referenced by key from the others.

| Area                          | Existing construction/consumer boundary                                                                                                                                                                                                                                |
| ----------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Hub scoped store adapters     | `hub/store/` constructors and factories reached from `build_hub_core`, `build_tenant_stores`, and `build_tenant_services`; shared scoped connection/transaction primitives                                                                                             |
| Finding exits and reads       | `FindingStore`, `FindingExitService`, `FindingBucketReader`; `hub/api/findings.py` and its finding/label read helpers                                                                                                                                                  |
| Configured adapters           | `build_live_config`, `build_config_authoring`, `build_hub_settings`, `build_hub_flags`, `ConfigApplication`, forge/work-source registries, budgets, commit resolver, secrets and config references; config/secrets/forge/credential/repository/work-source API modules |
| Hub identity                  | `hub/auth/` construction in `build_services` and `build_tenant_services`, including tenant invitations; auth/session/login, identity-provider, membership, self, hub-administrator and credential-auth dependencies                                                    |
| Projects and urgency          | `hub/domain/projects/`, `hub/domain/urgency/`; projects, project-label/parameter helpers, urgencies API                                                                                                                                                                |
| Graphs                        | `hub/domain/graph/`, graph stores/codecs and packaged graph inputs; system-artifact API and graph consumers                                                                                                                                                            |
| Fleet registration            | `hub/domain/runners/`, `FleetService`, `DetachService`, `HostRetirementService`; fleet/host/runner API families                                                                                                                                                        |
| Chunk execution               | Remaining `hub/domain/execution/` providers; chunk facts/questions/claim/apply and runner-contact handlers                                                                                                                                                             |
| Chunk operations              | `hub/domain/operations/`, `IngestService`, `DependencyService`, `WorkItemEditService`, event log; chunk edit/dependency/views/queue and ingest helpers                                                                                                                 |
| Garden authoring              | Scopes, routines, baselines, run context, answered findings, proposal authoring/closure and delivery resolution; scope/routine/proposal API                                                                                                                            |
| Garden materialization        | Finding sets, garden/review delivery recorders/materializers and formats; dependencies shared by the flow executor                                                                                                                                                     |
| Garden runs                   | Trend/sweep/run readers and services; garden-run API                                                                                                                                                                                                                   |
| Cadence                       | `hub/domain/cadence/`, cadence stores and errand planning; cadence API and tenant cadence                                                                                                                                                                              |
| Flow execution                | `hub/delivery/` tenant providers, `FlowLineFactory`, `FlowLineReconciler`, `build_flow_driver`, flow-place API                                                                                                                                                         |
| Observability                 | Transcript/analytics/event derivation, trace and egress providers, annotation, invariants; events/chunk-events/status API and exports                                                                                                                                  |
| Hub lifecycle and passes      | Process action loop/wake/pass driver, sweep roster, tenant administration/teardown, close drain, cadence/annotation/event sweeps, hosted-app wiring                                                                                                                    |
| Host infrastructure           | `_build_host_graph`, host stores, workspaces, subscriptions, host identity/readiness, shared clocks, children, telemetry and tracing                                                                                                                                   |
| Runner stores                 | `_build_stores`, bound connections and runner read/store factories                                                                                                                                                                                                     |
| Runner identity and harnesses | `_build_runner`, token/identity state and clients, executor, retained harness sets, runner tracing                                                                                                                                                                     |
| Runner loop collaborators     | `LoopWiring.context`, `with_context`, per-step collaborators and operation-owned resources                                                                                                                                                                             |
| Runner API                    | Served app, federation/readiness/telemetry and roster selection; named host/runner dependencies                                                                                                                                                                        |
| CLI                           | Each selected command under `cli/`, `hub/cli/`, `runner/cli/`; shared carriers and lazy groups                                                                                                                                                                         |

API functions sharing a module but belonging to different rows move only with their owner. Chunk lifecycle/read/view and
queue helpers belong to chunk operations; the `chunk_statuses` API and event/export status handlers belong to
observability. Shared helpers are narrowed to typed arguments when their first migrated caller needs them; remaining
callers can pass those values from a bag. Constructor ownership does not migrate twice.

Local CLI runtime consumers are narrowed before their runtime aggregate loses fields, even when that command's own
container migration is later. This includes host serving, runner/pass threads, scratch reconciliation, offline store
helpers, and `tools.invariants` callers. Bounded compatibility entrypoints retain their existing call signatures until
their command migrates; they obtain named capabilities from registered providers, not retired aggregate fields.

## Flows

```text
registered legacy leaf → adopted provider under the same key → no legacy construction
bag field with live consumers → named consumer dependencies → field and bridge removed
nonempty bag assembler → last consumer narrowed → assembler and type removed
process with legacy registrations → empty legacy registration → remove that module/root call
```

Every migration change checks both canonical object identity and behavior through its changed entrypoint. Available
retirement happens in that change (`decision:adopt-di/declared-cutover`); no builder is deleted while it still assembles
a bag needed by another consumer. A dedicated final closure change removes the remaining transition allowances and
proves there are no legacy owners left, rather than postponing available retirement to that change.

The inventory covers constructors in roots and helpers, route dependency functions, auth records, background callbacks,
and API helper objects, not just handler parameters. New services in an activated process enter their area's registry;
the inventory admits no new legacy construction (`engineering:born-in-container`).
