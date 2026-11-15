# Area registration

## System

```text
process root
  ├─ chooses configured adapter packages and calls their register methods
  ├─ passes typed adapter registrations to each area's register method
  └─ builds one validated container

area/wiring.py → foundation registry interface ← process translation → dishka
domain service → its constructor ports                         API edge → injection
```

| Owner           | Declares                                                    | Does not declare                         |
| --------------- | ----------------------------------------------------------- | ---------------------------------------- |
| Process root    | Areas, configured adapter choices, external inputs          | Individual domain services               |
| Adapter package | Concrete providers, lifetimes, owned resource finalizers    | Another adapter's providers              |
| Area wiring     | Services, explicit factories, aliases for its own ports     | Concrete adapters or their lifetimes     |
| Translation     | Dishka providers, validation, scope mapping, test overrides | Domain rules                             |
| Entrypoint edge | Typed resolution for its route, command, or driver          | A general resolver passed to domain code |

## Domain

```python
# foundation/: no dishka imports. Ref is an opaque covariant registration handle.
T_co = TypeVar("T_co", covariant=True)

@dataclass(frozen=True)
class Ref(Generic[T_co]):
    key: object

@dataclass(frozen=True)
class Port[T]:
    annotation: object  # the constructor annotation this port aliases

class Lifetime(Enum):
    PROCESS = "process"
    TENANT = "tenant"
    REQUEST = "request"
    RUNNER = "runner"
    COMMAND = "command"
    OPERATION = "operation"

class Registry(Protocol):
    def service[T](self, source: type[T], *, lifetime: Lifetime) -> Ref[T]: ...
    def factory[T](self, build: Callable[..., T], *, lifetime: Lifetime) -> Ref[T]: ...
    def resource[T](self, open: Callable[..., Iterator[T]], *, lifetime: Lifetime) -> Ref[T]: ...
    def context[T](self, port: Port[T], *, lifetime: Lifetime) -> Ref[T]: ...
    def alias[T](self, port: Port[T], target: Ref[T]) -> None: ...
```

| Operation  | Key and ownership                                                                                     |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| `service`  | Concrete class key; constructor parameters are resolved by their evaluated annotations                |
| `factory`  | Annotated return key; every parameter is annotated; defaults are passed explicitly inside the factory |
| `resource` | Yielded annotation key; one yield, finalizer owned by the declared lifetime                           |
| `context`  | Declared annotation key; supplied when that lifetime opens; borrowed, with no implicit finalizer      |
| `alias`    | Port annotation resolves the target key; no new construction, cache, or finalizer                     |

`Ref` is nominal, identifies a registration issued by the registry, and is not a live instance. A `Port[T]` constant
names a runtime annotation; the area's typed `Ref[PortType]` parameters check adapter compatibility before its aliases
are declared. Factories use normal Python annotations, not a parallel dependency-map language.

## Interfaces

```python
# Area-owned ports and wiring; engineering:area-wiring, engineering:one-instance-per-port.
READ_FINDINGS: Port[IReadFindingRepository] = Port(IReadFindingRepository)
WRITE_FINDINGS: Port[IWriteFindingRepository] = Port(IWriteFindingRepository)

def register(
    registry: Registry,
    *,
    findings: Ref[IWriteFindingRepository],
    finding_reads: Ref[IReadFindingRepository],
) -> None:
    registry.alias(WRITE_FINDINGS, findings)
    registry.alias(READ_FINDINGS, finding_reads)
    registry.service(FindingExitService, lifetime=Lifetime.TENANT)
    registry.service(FindingBucketReader, lifetime=Lifetime.TENANT)

# Adapter registration returns one handle usable at both structurally compatible ports.
def register_findings(registry: Registry) -> Ref[FindingStore]:
    return registry.service(FindingStore, lifetime=Lifetime.TENANT)

# Root calls areas, not individual service constructors.
finding_store = store_wiring.register_findings(registry)
finding_wiring.register(registry, findings=finding_store, finding_reads=finding_store)
```

```python
# Representative factory/default and resource forms; definitions here are illustrative types.
class ExampleService:
    def __init__(self, clock: IClock, limit: int = 32) -> None: ...

def example_service(clock: IClock) -> ExampleService:
    return ExampleService(clock=clock, limit=32)  # preserve the existing constructor's default

def owned_engine(settings: DatabaseSettings) -> Iterator[Engine]:
    engine = create_engine(settings.url)
    try:
        yield engine
    finally:
        engine.dispose()

registry.factory(example_service, lifetime=Lifetime.PROCESS)
registry.resource(owned_engine, lifetime=Lifetime.PROCESS)
registry.context(Port[StoreScope](StoreScope), lifetime=Lifetime.TENANT)
```

```python
# engineering:role-is-a-type. Distinct constructor annotations, even for one underlying type.
class WallClock(IClock, Protocol): ...
class ScheduleClock(IClock, Protocol): ...

def throttle(clock: WallClock) -> Throttle: ...
def schedule(clock: ScheduleClock) -> Schedule: ...
```

Each role has a distinct registration key. Existing public constructor seams retain their annotations unless a
same-type/different-role collision requires narrowing; the distinction does not wrap every service or clock.

```python
# Translation/test edge only; never part of Registry.
class TestOverrides(Protocol):
    def replace_factory[T](self, target: Ref[T], build: Callable[..., T]) -> None: ...
    def replace_value[T](self, target: Ref[T], value: T) -> None: ...
```

Overrides require an existing owned target and retain its key/lifetime; aliases keep targeting that key. A supplied
replacement value is borrowed. Replacement happens before validation, not while a container is running.

| Translation boundary | Contract                                                                                                                                                                                               |
| -------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Production build     | Pin a non-deprecated Dishka 1.x API; evaluate annotations; use strict validation; reject duplicate keys, missing dependencies, cycles, and longer-lived providers requiring shorter-lived dependencies |
| Defaults             | Reject automatic registration of a constructor carrying defaults; require an explicit factory                                                                                                          |
| Alias                | Alias the owned target registration; reject a second owner, never infer a new adapter instance                                                                                                         |
| Scope values         | Map `Port.annotation` to Dishka context dependencies; missing values fail resolution                                                                                                                   |
| Typing               | Keep the library's strict waiver inside translation; area modules and root registration calls remain strict-clean                                                                                      |
| Overrides            | A separate test-only builder replaces an existing provider before validation; area `Registry` has no replacement operation                                                                             |

Runtime container types and generic resolution stay in translation and entrypoint adapters. Domain services have no
`Ref`, container, or request-state parameter. Wiring modules may import their own package's ports and service types,
never another area's wiring or concrete adapters (`engineering:dishka-at-edge`, `engineering:wiring-modules-apart`).
