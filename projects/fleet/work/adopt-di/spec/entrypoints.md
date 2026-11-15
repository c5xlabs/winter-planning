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

async def tenant_request(
    request: Request,
    admitted: Annotated[AdmittedTenant, Depends(resolve_tenant)],
) -> AsyncIterator[RequestContainer]:
    owned = tenant_containers.request(admitted, request_metadata(request))
    request.state.dishka_container = owned.resolver
    try:
        yield owned.resolver
    finally:
        owned.close()

# Router dependency; inject resolves arguments only after FastAPI has resolved dependencies.
router = APIRouter(dependencies=[Depends(reject_runner_principal), Depends(tenant_request)])

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

`FromDishka` resolution is sequenced after the admission dependency. The API integration attaches the selected request
child as the resolver used by injection; stock middleware that creates an unrelated child from the app root is not used
for tenant routes. The request child contains typed principal and metadata values; route authorization remains explicit.
Both synchronous and asynchronous handlers use `inject_sync` with synchronous `get()`; ordinary `inject`'s
async-container resolution is not used.

Hub-only routes attach the borrowed process resolver and use FastAPI's request-local authentication dependency. They
neither open a tenant nor put a principal in the process container. Unknown tenant, closed tenant, credential/header
mismatch, missing identity, and membership/admission refusals retain their current status and payload.

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

| Command boundary           | Registration/import rule                                                                                                                        |
| -------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------- |
| Operator client verbs      | Client/context/trace registrations only; no daemon store or app imports                                                                         |
| Developer invariant check  | Client mode loads client registrations; explicit offline hub/runner mode loads only the selected store/invariant providers after mode selection |
| Hub tree                   | Selected client verb or selected local runtime/store verb; runtime providers imported inside that selected module                               |
| Runner worker verbs        | Worker-session, launcher, transport, and client registrations; never import `runner.cli.runtime` or daemon registrations                        |
| Runner local/runtime verbs | Only the selected host, tick, ingest, migration, check, or offline-read graph                                                                   |
| Daemon-start command       | Command hands process-owned resources to the hosted runtime; command exit does not close the running graph twice                                |

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
