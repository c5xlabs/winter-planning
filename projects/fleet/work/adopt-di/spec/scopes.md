# Scope ownership

## System

| Graph               | Lifetime path                                                                                                     | Owner                                                                               |
| ------------------- | ----------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Hub tenant work     | process → tenant → request                                                                                        | Hub runtime → tenant cache → request integration                                    |
| Hub-only request    | process services plus request-owned auth context                                                                  | Request integration borrows the process resolver; no artificial tenant              |
| Hub background work | process → cached tenant                                                                                           | Driver's typed tenant accessor; no request or synthetic principal                   |
| Runner host         | process → runner → operation, when operation-local inputs are needed                                              | Host runtime → `HostRunnerSet` → driver/invocation                                  |
| CLI                 | command; a local or offline command's owning-process root; or the hosted process graph for a daemon-start command | Selected command's context manager; daemon startup transfers to its process runtime |

| Lifetime    | Values and collaborators                                                                                                                                                                                             | Closure                                         |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| Hub process | Borrowed store, tracing and settings; hub-level stores and identity services, `TenantBrokers` directory, action loop, tenant cache                                                                                   | After request/driver shutdown and child closure |
| Tenant      | `StoreScope`, `ScopedConnections`, stores, tenant services, `TenantBrokers.for_tenant(id)`, configured adapter state                                                                                                 | Eviction or hub shutdown                        |
| Request     | Principal, request metadata, genuinely request-local providers                                                                                                                                                       | Response/stream completion or cancellation      |
| Host        | Engine, host stores, shared clocks, host identity and telemetry; `HostSettingsReader`, `HostRecordsReconcile` and the holders it sets (`HostWorkspaces`, `HostTickInForce`), the sealed-token store, `HostRunnerSet` | After serving threads and runners stop          |
| Runner      | Bound connections/stores, identity/token holders, its `RunnerSettingsReader`, executor, retained harness sets, the runner's stepper                                                                                  | After loop/resume work drains                   |
| Operation   | Borrowed caller-thread clients, selected harness-set snapshot, per-operation tracing context                                                                                                                         | After the operation; only owned resources close |
| Command     | Carrier overrides, selected client resources, command-local services of a client-only command                                                                                                                        | Command exit, including exceptions              |

The hub's engine, `HubStoreConnections`, platform tracing and `HubSettingsGraph` are not container resources. `hub host`
opens them once (`open_hub_store`, then `build_hub_settings`), supplies them to the hub root as borrowed context values,
and disposes them after the hosted runtime stops; no provider acquires a hub store or closes these values. This keeps
one traced store per hub process, the shape `test_hub_store_wiring_goes_through_composition` guards.

The tenant broker is never process-scoped: only its directory is shared. `ScopedConnections` is an acquiring seam, not a
connection checked out for a tenant or request lifetime. Store transactions keep their existing boundaries.

### Objects built from settings

Under `epic:config/runtime`, an object built from settings is rebuilt when the revisions it was built from move, at the
start of a pass (`bzh:settings-snapshot-per-pass`, `bzh:config-read-on-use`). The container never holds such an object.
It builds the stable owner that holds it, and the owner rebuilds it:

| Owner (container lifetime)         | What it rebuilds, and when                                                                                                                                                                                                                                                                                              |
| ---------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `RunnerStepper` (runner)           | The `LoopContext` and its hub client, when the host or runner revision, the harness key, or the timeout moves                                                                                                                                                                                                           |
| The runner's harness sets (runner) | The harness set, when the `session` keys it reads move                                                                                                                                                                                                                                                                  |
| `HostRecordsReconcile` (host)      | The `WorkspaceRegistry` it sets in `HostWorkspaces`, when a workspace record moves, unmoved workspaces keeping their providers and locks kept by `workspace_id`; each subscription's sampler and renewer, when its record moves, and with them the `HostTick` and the served subscriptions it sets in `HostTickInForce` |
| `HostRunnerSet` (host)             | Its runner children, when a hosted-runner record is added or retired (§Flows)                                                                                                                                                                                                                                           |

An owner builds its first object when its child opens, from the settings in force at that start, so a bad entry fails
the start step that reports it today (`compose` or `serve` in the host's runner report); every later pass rebuilds from
that pass's snapshot. No provider captures one pass's settings into a longer-lived object: a provider takes the settings
reader, and its owner reads per pass. The owner keeps the last good object when a rebuild fails, as
`epic:config/runtime` specifies. `HostTick` is itself a rebuilt object: a moved subscription record replaces it, with
its sample and renewal passes, and a host with no subscriptions runs it with no sample pass, as today. The container
holds the `HostTickInForce` holder, never a `HostTick`, and the host pass reads the tick from the holder each pass.
`HostRecordsReconcile` is registered `optional`, `None` for a host keeping no records, as today. A runner's
`RunnerSettingsReader` reads its stored entries on use and rebuilds nothing; the runner child builds it, at runner
lifetime, over its own settings store and identity. The per-runner seed builds a seed-time reader of its own, as today,
because it runs before the child opens and without the child's identity. `RunnerStepper` also carries retention state
across a rebuild and closes a replaced hub client only once it is idle. A new settings-built object follows this table
rather than a new container lifetime: it gets a stable owner registered at the owner's lifetime.

## Interfaces

```python
# Translation/entrypoint-only interfaces; engineering:locked-child-container. As shipped in foundation/registry/scopes.py.
def open_child(parent: OwnedChild, *, lifetime: Lifetime, owner: str, values: ScopeValues) -> OwnedChild: ...

class OwnedChild(Protocol):
    @property
    def lifetime(self) -> Lifetime: ...
    @property
    def owner(self) -> str: ...  # a tenant id, a runner id, or a process label
    @property
    def resolver(self) -> Resolver: ...
    def close(self, exception: BaseException | None = None) -> None: ...

class TenantContainers(Protocol):
    def resolve[T](self, scope: StoreScope, service: type[T]) -> T: ...
    def open(self, scope: StoreScope) -> None: ...  # cache lookup/open; raises UnknownTenant or TenantClosed
    def every_open[T](self, service: type[T]) -> Iterator[tuple[TenantId, T]]: ...  # replaces HubCore.every_open
    def request(self, admitted: AdmittedTenant, metadata: RequestMetadata) -> OwnedRequest: ...
    def evict(self, tenant: TenantId) -> None: ...
    def close(self) -> None: ...
```

`OwnedChild` owns only the child it opened; `OwnedRequest` specializes it for a request resolver. `ScopeValues` is a
collection of typed context-key/value pairs checked against the registry; it is not a service bag. `AdmittedTenant`
carries the already-resolved principal, tenant scope, and name, with no services field.

The container handles above are guarded synchronous resolvers, not raw Dishka containers. `get()` holds the owners'
state guards in ancestor-to-child order through resolution and refuses a closing/closed owner anywhere on that path,
raising `ScopeClosed`. A state guard is shared for resolution and exclusive for closing, so concurrent resolutions never
serialize on a parent's guard. Dishka itself holds each container's own lock for every `get()`, a cache hit included, so
lookups through one container serialize briefly on that lock; a request child reaches a tenant service through the
tenant container's lock. Lookups are short and construction is once per lifetime, so no cache sits in front of it.
Closing waits for active resolutions under those guards, marks the owner closing, then finalizes outside the guards and
the cache lock; existing request children cannot reconstruct resources through a closed parent. This serializes lookup
with closure, not the subsequent service operation, and adds no request-drain policy. Raw containers stay private to
translation and the single child-opening helper.

The injection-facing resolver's synchronous `get(dependency_type, component=None)` takes `component` as a
positional-or-keyword parameter, matching dishka's `Container.get`, because dishka's compiled injector passes it
positionally; it forwards it with the runtime annotation key; guarding does not alter dependency-key selection.

| Entry operation   | Contract                                                                                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `open_child`      | Only child-container constructor; explicit lifetime mapping; `lock_factory=threading.Lock`; never closes a borrowed parent                                   |
| Tenant cache miss | Hold the cache's creation lock, confirm tenant exists/open, create one locked child with `StoreScope`, initialize required legacy bridge state, then publish |
| Tenant cache hit  | Return the existing child; tenant-specific services remain cached and shared by HTTP and background callers                                                  |
| First resolution  | Each container's own lock serializes provider construction, independently of the cache creation lock                                                         |
| Request           | Open one child beneath the admitted tenant; supply principal/request metadata there, never in a parent                                                       |
| Runner            | `HostRunnerSet` opens one locked child per runner with its context values, before resolving stores; host-scoped stores remain in the parent                  |
| Operation         | Supply borrowed caller-thread values or factories; never cache a selected operation's principal/client/set snapshot in a parent                              |

A runner child's one context value is its `BuiltRunner` (its hub-minted id, the start snapshot of its settings, its
token, and its hosted-runner entry); `open_child`'s `owner` is that id, which `ScopeClosed` carries. Everything else the
runner needs is resolved beneath it.

The hub maps `PROCESS`, `TENANT`, and `REQUEST` to consecutive non-skipped Dishka scopes. The host separately maps
`PROCESS`, `RUNNER`, and `OPERATION`; a command maps `COMMAND`. Unused lifetime kinds are rejected by that process's
translation, not silently mapped to a different lifetime.

A local or offline command that needs a hub or host graph (an offline store mode, a dual-mode verb's store path, a local
runtime verb, the standalone `tick`) builds that owning process's translation, hub `PROCESS`/`TENANT` or host
`PROCESS`/`RUNNER`, as a root the command owns: it opens the tenants or runners it selects through the same owners, and
closes the root on exit, including exceptions. Such a command builds that one container and no other: its own
command-local services (carrier overrides, its selected clients) register at that translation's `PROCESS` lifetime,
because for the command's duration the command is the process, and whole-graph validation covers the one container.
`COMMAND` is the lifetime of a client-only command, which has no owning process graph; a command never registers another
process's area services at `COMMAND`.

The generic guarded resolver raises `ScopeClosed(lifetime, owner)`, carrying the closed owner's lifetime and its id (a
tenant id or a runner id), so each handler matches only its own kind of closure. The hub's tenant owner converts a
`ScopeClosed` raised through a closed tenant into `TenantClosed`, so the hub's existing handler answers it; the host's
runner owner leaves it as `ScopeClosed` for the runner app's handler.

## Flows

```text
tenant cache: absent → opening under creation lock → ready → removed/closing → closed
                  └─ construction failure → close acquired resources → absent; rethrow

request: admitted → open locked request child → resolve named providers → finish/cancel → close request child
                   parent tenant remains cached

runner start: open runner child → build the runner from it → startup steps → place in HostRunnerSet → wire/serve/drive
                  ├─ fails before placement → close runner child → held in failures → next reconcile opens a new child
                  └─ fails after placement → abandon: withdraw, remove, stop drivers, close runner child,
                     close wiring → same retry

host pass: host settings snapshot → HostRecordsReconcile (workspaces, then subscriptions)
             → retirement notices → HostRunnerSet reconcile → the HostTickInForce tick → host report
host boot: host settings snapshot → HostRecordsReconcile → runner children open

runner drain: withdraw from HostRunnerRegistry → stop drivers → quiesce → on_shutdown → close runner child
                  → close wiring → remove from HostRunnerSet → drop sealed token
```

The records reconcile runs before any runner starts, resumes or reconciles, in a pass and at boot alike, so a retired
workspace's provider releases what it holds first; this is the order `epic:config/runtime` delivered.

The runner child holds the executor, the stepper and the per-runner hub clients, and closes them in reverse construction
order, so the stepper and its clients close before the executor they were built over. "Close wiring" then closes only
what the served runner holds outside the child: its served registration, its drivers' threads and `_WiredRunner`'s own
handles.

| Failure or shutdown                        | Owned flow                                                                                                                                                                                                                                                        |
| ------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Provider fails before returning a resource | That factory unwinds its own partial acquisition; completed resource providers remain owned by the enclosing scope                                                                                                                                                |
| Tenant initialization fails                | Do not publish the cache entry; unwind acquired resources and rethrow the original failure, retaining cleanup failures as context                                                                                                                                 |
| Later lazy resolution fails                | Propagate the failure; do not add an automatic retry or abandon the scope's acquired resources                                                                                                                                                                    |
| Tenant closes                              | Existing deletion closes tenant admission first; remove cache entry under its lock, then drop that tenant's broker and close its configured resources outside the cache lock                                                                                      |
| In-flight work after tenant close          | Existing scoped-store closed-tenant refusals and stream closure still apply; a resolution refused by the closed tenant raises `TenantClosed`, which `refuse_closed_tenant` answers as today (`decision:adopt-di/closed-tenant-request`)                           |
| Request fails/cancels                      | Close request-owned resources; never close the cached tenant or shared engine                                                                                                                                                                                     |
| Hub stops                                  | Stop admission and producers according to existing app lifecycle; close configured tenant clients while the action loop runs; close remaining tenant scopes; stop the loop; leave the borrowed store and tracing for `hub host` to dispose                        |
| Settings rebuild fails                     | The owner keeps serving the last good object, records the failure, and retries at the next pass; a first build that fails propagates                                                                                                                              |
| Runner drains (retire or host tick)        | The existing `drain_runner` order: the runner child closes, then the wiring; `abandon` adopts the same order; sibling runners and the host are untouched                                                                                                          |
| Host stops                                 | Stop serving threads; `HostRunnerSet.stop_all` stops drivers, waits out passes and marks workers; per runner, close its child, then its wiring, the drain order, in place of today's wiring-first `close_all`; then host-owned clients/store engine and telemetry |
| Standalone runner (`closes_host`)          | One runner on its own host; closing that runner also closes the host container, as `closes_host` does today                                                                                                                                                       |
| Borrowed resource                          | Its supplying owner closes it; no resource finalizer is registered for the borrowed value                                                                                                                                                                         |

`HostRunnerSet` is the only owner of runner children: it opens one through `open_child` and closes it, and no other
caller does. A failed child is never reused; every start attempt, a reconcile retry included, opens a new one. This
preserves today's behavior, where each attempt calls `compose_runner` again, and mirrors a failed tenant open publishing
nothing.

Cached owners enumerate and close their children; closing a Dishka parent is not assumed to close independently cached
children. Owner adapters reject lookup after closure and make explicit close idempotent, including partially initialized
owners. Cleanup attempts all owned finalizers; cleanup errors do not skip sibling closure or mask an active primary
failure.

Thread-confined resources are constructed and closed on their owning thread/loop through the existing owner. The
container receives a borrowed value, an owner-dispatching factory, or a synchronous finalizer that joins the existing
owner's close operation before that owner stops (`decision:adopt-di/synchronous-resolution`). Async service methods do
not make construction async; no event-loop lock is shared between the API and action loop.
