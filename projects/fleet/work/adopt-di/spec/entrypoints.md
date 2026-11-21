# Entrypoint resolution

## Interfaces

```python
# API edge. The existing auth helpers take named dependencies instead of HubServices.
from dishka.integrations.fastapi import FromDishka, inject_sync

@dataclass(frozen=True)
class AdmittedTenant:
    identity: Principal
    scope: StoreScope
    name: str

def tenant_request(
    request: Request,
    admitted: Annotated[AdmittedTenant, Depends(resolve_tenant)],
) -> Iterator[RequestContainer]:
    owned = tenant_containers.request(admitted, request_metadata(request))
    request.state.dishka_container = owned.resolver
    try:
        yield owned.resolver
    finally:
        owned.close()

def process_request(request: Request) -> None:
    request.state.dishka_container = process_resolver  # borrowed; the request does not close it

# Router dependencies run in order; inject resolves arguments only after FastAPI has resolved dependencies.
router = APIRouter(
    dependencies=[
        Depends(process_request),
        Depends(reject_runner_principal),
        Depends(tenant_request),
        Depends(tenant_bound),
    ],
)

# Representative dependency parameters; existing wire/body parameters are omitted.
@router.post("/findings/resolve", response_model=list[FindingView])
@inject_sync
def resolve_findings(
    request: FindingExitRequest,
    exits: FromDishka[FindingExitService],
    findings: FromDishka[IReadFindingRepository],
    identity: Annotated[ResolvedIdentity, Depends(require(Permission.CHUNK_CONTROL))],
) -> list[FindingView]: ...
```

Tenant routes resolve in a fixed sequence. `process_request` attaches the borrowed process resolver first; the auth
dependencies that run before admission (`reject_runner_principal`, `resolve_tenant`, `require`, `granted`) resolve the
named process services they read from it, instead of taking the `Hub` bag. `resolve_tenant` maps `UnknownTenant` and
`TenantClosed` from `TenantContainers.open` to today's `404`, as `_served` does now: the tenant opens in
`resolve_tenant`, so a failed open answers `404` to every caller, a runner included. `tenant_request` then replaces the
request's resolver with the admitted tenant's request child. `tenant_bound` stays after it, binding the log context and
tenant span as today, and `_route_admission`'s scan for `TenantBoundAdmitting` still finds it among the route's
dependencies. `resolve_tenant` and `tenant_request` are synchronous dependencies, so a cache miss's tenant open runs in
FastAPI's threadpool, as it does today, never on the event loop. Provider construction on first resolution runs where
the handler resolves: in the threadpool for a synchronous handler. An asynchronous handler (`apply_config`,
`events_stream`) does not take a lazily built tenant service through `FromDishka`; it takes it from a synchronous
dependency, which FastAPI runs in the threadpool.

`FromDishka` resolution is sequenced after the admission dependency. The API integration attaches the selected request
child as the resolver used by injection; stock middleware that creates an unrelated child from the app root is not used
for tenant routes. The request child contains typed principal and metadata values; route authorization remains explicit.
Both synchronous and asynchronous handlers use `inject_sync` with synchronous `get()`; ordinary `inject`'s
async-container resolution is not used. On the hub and the runner app alike, an asynchronous handler takes through
`FromDishka` only process- or host-lifetime services the root builds eagerly at startup; anything lazily built, or
anything reached through a child container's lock, it takes from a synchronous dependency, so `get()` never waits on
another thread's construction on the event loop.

Hub-only routes use `process_request` alone and FastAPI's request-local authentication dependency. They neither open a
tenant nor put a principal in the process container. Unknown tenant, closed tenant, credential/header mismatch, missing
identity, and membership/admission refusals retain their current status and payload.

```python
# Runner API edge. HostRunnerRegistry entries hold each served runner's child container in place of ServedRunner.seams.
def bind_runner(request: Request, runner_id: Annotated[str, Path()]) -> None:
    served = registry.current().resolve(runner_id)  # 404 unknown, 503 halted, as today
    request.state.dishka_container = served.container

router = APIRouter(prefix="/runners/{runner_id}/api", dependencies=[Depends(bind_runner)])

@router.get("/dashboard")
@inject_sync
def dashboard(status: FromDishka[RunnerStatusService], jwks: FromDishka[IJwksCache | None]) -> DashboardView: ...
```

```python
# Host routes that report on every runner take this typed accessor, not RunnerWiring.for_runner.
class ServedRunners(Protocol):
    def each[T](self, service: type[T]) -> Iterator[tuple[str, T]]: ...  # skips a runner whose child is closing
```

Runner routes resolve from the selected runner's child container; host routes resolve from the host container, which
`host_request`, the runner app's counterpart to `process_request`, attaches as a router dependency. A host route that
reports per runner (environments, subscriptions, the runners list, hub tokens) iterates `ServedRunners`, which
`HostRunnerRegistry` provides from the same `current()` read; a runner whose child closes mid-iteration is left out of
the response, as a runner withdrawn before the read is today, and never fails the request. `bind_runner` reads
`HostRunnerRegistry.current()` once per request, as today, so a runner served or withdrawn mid-flight is either fully
resolvable or answers `404`. A resolution refused because the runner's child closed mid-request raises `ScopeClosed`,
which an app-level exception handler on the runner app, matching only a closed runner child, answers with the same `404`
a withdrawn runner gets (`decision:adopt-di/drained-runner-request`). `RunnerWiring` and `ServedRunner.seams` are
retired.

Three readers of the served runners run outside any router, in the runner app's middleware and on the event loop:
`_served_identity` (the identity-stamping middleware), `_FirstServedIdentity`, and `_gate_web_surface`'s
`require_human_session`. They read `HostRunnerRegistry.current()`, the runner child's context values (runner id,
`BuiltRunner`), its `ICurrentRunnerIdentity` holder, which the child builds eagerly when it opens, or host services the
root builds eagerly; they never construct lazily, and a runner whose child is closing is skipped, so a `ScopeClosed`
cannot escape middleware as a `500`.

A service the app may be built without (the OpenAPI export build `create_app_for_export`, partial test apps) has an
explicit absent registration (`Registry.absent`, registered by the area that owns the service, in the root that builds
the app without it), so whole-graph validation passes. A route takes the service as plain `FromDishka[T]`; resolving an
absent registration raises `ServiceAbsent(label)` before the handler runs, and an app-level exception handler answers
it: on the runner app with the `503` `RunnerWiring` gives today (`"{label} not wired — start via …"`, same status and
payload), on the hub with `hub_store_unavailable_problem`'s `503`. Each absent registration carries its accessor's
existing label, so a label shared by several services today, such as `runner store`, stays shared; the hub's store-free
app registers its store-backed services absent with the `hub_store_unavailable` label.

A read that degrades instead of refusing (the runner's `maybe_` accessors, such as `maybe_status`,
`maybe_hosted_runners` and `maybe_start_failures`; the hub's disabled trace and egress exports) takes
`FromDishka[T |
None]`, registered with `Registry.optional` by the area that owns `T`: the instance when the root has
it, `None` when the root does not, and the route keeps degrading as today. A service that is both refused by some routes
and degraded by others has both registrations over the one owner.

```python
# Domain driver receives this typed seam, not TenantContainers or a generic get().
type CloseDrainerFor = Callable[[StoreScope], CloseIntentDrainer]

# Entrypoint factory supplies the seam from the cached tenant graph.
def close_drainer_for(scope: StoreScope) -> CloseIntentDrainer:
    return tenant_containers.resolve(scope, CloseIntentDrainer)
```

The same pattern replaces `TenantServicesOpener` callbacks in flow, cadence, event derivation, annotation, boot
recovery, and tenant registry/host lookups. A callback names the particular service or read port; no domain object
accepts a generic resolver or chooses arbitrary registration keys.

```text
selected CLI command
  → load its command module through existing LazyGroup
  → read invoker's CliCollaborators / command carrier
  → load only that command's area registrations
  → apply typed carrier values as test/production inputs
  → build and validate command container
  → resolve command's named collaborators
  → execute
  → close command-owned resources
```

| Command boundary            | Registration/import rule                                                                                                                                                                                                                                                                                                                 |
| --------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Operator client verbs       | Client/context/trace registrations only; no daemon store or app imports                                                                                                                                                                                                                                                                  |
| Developer invariant check   | Client mode loads client registrations; explicit offline hub/runner mode loads only the selected store/invariant providers after mode selection                                                                                                                                                                                          |
| Hub tree                    | Selected client verb or selected local runtime/store verb; runtime providers imported inside that selected module                                                                                                                                                                                                                        |
| Runner worker verbs         | `heartbeat`, `session-end`, `ask`, `attach`, `work-items`, `pm-items`, `chunk`, `artifact`, `garden`, `finding`, `scope`, `analytics`, `opencode drive` and `attend` (every verb calling through `WorkerCall`): worker-session, launcher, transport, and client registrations; never import `runner.cli.runtime` or daemon registrations |
| Runner daemon-client verbs  | Client registrations only, reaching the running host through `RunnerDaemon`: control, settings, traces, `transcript list` and `get`, `prompt clear`, the workspace and subscription record verbs, apply and export                                                                                                                       |
| Runner dual-mode verbs      | `tick`, `add`, `retire`, `token set`: the client path while a host answers; otherwise the selected store graph under `store_write_guard`                                                                                                                                                                                                 |
| Runner local/runtime verbs  | Only the selected graph: `host`, `init`, `migrate`, `ingest`, `store reset`, `transcript backfill` and `reship` (store writes under `store_write_guard`), `prompt list`, `get`, `install`, `diff` and `status` (host file and offline store reads), `harness status`, `external-usage probe`, `opencode compatibility`                   |
| Hub daemon-start command    | `hub host` opens the store, tracing and settings, lends them to the hosted root as context, and disposes them after the runtime stops; no double close                                                                                                                                                                                   |
| Runner daemon-start command | `runner host` builds the host root, which owns its engine and closes it with the host container, as `HostProcess.close` does today                                                                                                                                                                                                       |

A group may select or pass inputs but does not import/register the union of all its commands' providers. Carrier fakes
supplied with `CliRunner.invoke(obj=...)` keep precedence, including HTTP clients, launcher, transports, process
builders, and harness-bundle builders. A daemon-start command builds the hosted process root directly with command
inputs rather than constructing a duplicate command-owned process graph. Delayed provider construction does not
establish import isolation.

The first command-container path is `blizzard hub status`, using shared client/context/trace registrations. Other
commands retain their existing construction until their selected-command paths migrate; installing shared support does
not activate or assemble sibling command graphs.

`blizzard dev check-invariants` retains both running-daemon client mode and explicit offline hub/runner store mode.
Offline mode may load daemon store adapters, but neither client mode nor loading unrelated operator commands imports
those offline providers. The offline root owns its selected engines and closes them on success or failure; existing
external-database confirmation, option exclusions, after-recovery behavior, output, and exit status stay unchanged.

## Flows

```text
HTTP request
  → resolve machine credential or user identity with process services
  → existing tenant selection and route-specific admission
  → tenant cache lookup/open
  → request child with principal and metadata
  → required permission check + named route dependencies
  → service operation
  → response/stream finishes
  → request child closes

background corpus row
  → select row's tenant using existing pass rules
  → typed tenant accessor
  → same cached tenant service used by HTTP
  → operation, with existing per-tenant failure isolation
```

No route signature, command surface, config field, or wire payload changes as a consequence of dependency resolution
(`product:no-visible-change`).
