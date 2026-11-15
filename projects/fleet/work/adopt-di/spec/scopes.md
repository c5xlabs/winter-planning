# Scope ownership

## System

| Graph               | Lifetime path                                                        | Owner                                                                               |
| ------------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| Hub tenant work     | process → tenant → request                                           | Hub runtime → tenant cache → request integration                                    |
| Hub-only request    | process services plus request-owned auth context                     | Request integration borrows the process resolver; no artificial tenant              |
| Hub background work | process → cached tenant                                              | Driver's typed tenant accessor; no request or synthetic principal                   |
| Runner host         | process → runner → operation, when operation-local inputs are needed | Host runtime → runner roster → driver/invocation                                    |
| CLI                 | command, or the hosted process graph for a daemon-start command      | Selected command's context manager; daemon startup transfers to its process runtime |

| Lifetime    | Values and collaborators                                                                                             | Closure                                         |
| ----------- | -------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------- |
| Hub process | Engine, hub-level stores and identity services, `TenantBrokers` directory, action loop, tenant cache                 | After request/driver shutdown and child closure |
| Tenant      | `StoreScope`, `ScopedConnections`, stores, tenant services, `TenantBrokers.for_tenant(id)`, configured adapter state | Eviction or hub shutdown                        |
| Request     | Principal, request metadata, genuinely request-local providers                                                       | Response/stream completion or cancellation      |
| Host        | Engine, host stores, shared clocks/workspaces, host identity and telemetry                                           | After serving threads and runners stop          |
| Runner      | Bound connections/stores, identity/token holders, executor, retained harness sets                                    | After loop/resume work drains                   |
| Operation   | Borrowed caller-thread clients, selected harness-set snapshot, per-operation tracing context                         | After the operation; only owned resources close |
| Command     | Carrier overrides, selected client/store resources, command-local services                                           | Command exit, including exceptions              |

The tenant broker is never process-scoped: only its directory is shared. `ScopedConnections` is an acquiring seam, not a
connection checked out for a tenant or request lifetime. Store transactions keep their existing boundaries.

## Interfaces

```python
# Translation/entrypoint-only interfaces; engineering:locked-child-container.
def open_child(
    parent: ProcessContainer | TenantContainer | RunnerContainer,
    *,
    lifetime: Lifetime,
    values: ScopeValues,
) -> OwnedChild: ...

class OwnedChild(Protocol):
    @property
    def resolver(self) -> ProcessContainer | TenantContainer | RunnerContainer | RequestContainer: ...
    def close(self) -> None: ...

class TenantContainers(Protocol):
    def resolve[T](self, scope: StoreScope, service: type[T]) -> T: ...
    def request(self, admitted: AdmittedTenant, metadata: RequestMetadata) -> OwnedRequest: ...
    def evict(self, tenant: TenantId) -> None: ...
    def close(self) -> None: ...
```

`OwnedChild` owns only the child it opened; `OwnedRequest` specializes it for a request resolver. `ScopeValues` is a
collection of typed context-key/value pairs checked against the registry; it is not a service bag. `AdmittedTenant`
carries the already-resolved principal, tenant scope, and name, with no services field.

The container handles above are guarded synchronous resolvers, not raw Dishka containers. `get()` holds the owners'
state guards in ancestor-to-child order through resolution and refuses a closing/closed owner anywhere on that path.
Closing waits for active resolutions under those guards, marks the owner closing, then finalizes outside the guards and
the cache lock; existing request children cannot reconstruct resources through a closed parent. This serializes lookup
with closure, not the subsequent service operation, and adds no request-drain policy. Raw containers stay private to
translation and the single child-opening helper.

The injection-facing resolver's synchronous `get(dependency_type, component=...)` accepts and forwards Dishka's
`component` keyword as well as its runtime annotation key; guarding does not alter dependency-key selection.

| Entry operation   | Contract                                                                                                                                                     |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `open_child`      | Only child-container constructor; explicit lifetime mapping; `lock_factory=threading.Lock`; never closes a borrowed parent                                   |
| Tenant cache miss | Hold the cache's creation lock, confirm tenant exists/open, create one locked child with `StoreScope`, initialize required legacy bridge state, then publish |
| Tenant cache hit  | Return the existing child; tenant-specific services remain cached and shared by HTTP and background callers                                                  |
| First resolution  | Each container's own lock serializes provider construction, independently of the cache creation lock                                                         |
| Request           | Open one child beneath the admitted tenant; supply principal/request metadata there, never in a parent                                                       |
| Runner            | Bind the configured runner id before resolving stores; one locked child per runner; host-scoped stores remain in the parent                                  |
| Operation         | Supply borrowed caller-thread values or factories; never cache a selected operation's principal/client/set snapshot in a parent                              |

The hub maps `PROCESS`, `TENANT`, and `REQUEST` to consecutive non-skipped Dishka scopes. The host separately maps
`PROCESS`, `RUNNER`, and `OPERATION`; a command maps `COMMAND`. Unused lifetime kinds are rejected by that process's
translation, not silently mapped to a different lifetime.

## Flows

```text
tenant cache: absent → opening under creation lock → ready → removed/closing → closed
                  └─ construction failure → close acquired resources → absent; rethrow

request: admitted → open locked request child → resolve named providers → finish/cancel → close request child
                   parent tenant remains cached
```

| Failure or shutdown                        | Owned flow                                                                                                                                                                                                 |
| ------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Provider fails before returning a resource | That factory unwinds its own partial acquisition; completed resource providers remain owned by the enclosing scope                                                                                         |
| Tenant initialization fails                | Do not publish the cache entry; unwind acquired resources and rethrow the original failure, retaining cleanup failures as context                                                                          |
| Later lazy resolution fails                | Propagate the failure; do not add an automatic retry or abandon the scope's acquired resources                                                                                                             |
| Tenant closes                              | Existing deletion closes tenant admission first; remove cache entry under its lock, then drop that tenant's broker and close its configured resources outside the cache lock                               |
| In-flight work after tenant close          | Existing scoped-store closed-tenant refusals and stream closure still apply; no new request-drain or transaction policy                                                                                    |
| Request fails/cancels                      | Close request-owned resources; never close the cached tenant or shared engine                                                                                                                              |
| Hub stops                                  | Stop admission and producers according to existing app lifecycle; close configured tenant clients while the action loop runs; close remaining tenant scopes; stop the loop; dispose shared store resources |
| Runner stops                               | Stop its loop and drain resume/spawn work; wait for executor; close identity client; clear retained harness sets; close remaining runner resources                                                         |
| Host stops                                 | Stop serving threads; close all runners, then host-owned clients/store engine and telemetry using existing dependencies                                                                                    |
| Borrowed resource                          | Its supplying owner closes it; no resource finalizer is registered for the borrowed value                                                                                                                  |

Cached owners enumerate and close their children; closing a Dishka parent is not assumed to close independently cached
children. Owner adapters reject lookup after closure and make explicit close idempotent, including partially initialized
owners. Cleanup attempts all owned finalizers; cleanup errors do not skip sibling closure or mask an active primary
failure.

Thread-confined resources are constructed and closed on their owning thread/loop through the existing owner. The
container receives a borrowed value, an owner-dispatching factory, or a synchronous finalizer that joins the existing
owner's close operation before that owner stops (`decision:adopt-di/synchronous-resolution`). Async service methods do
not make construction async; no event-loop lock is shared between the API and action loop.
