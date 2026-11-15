# Adopt dependency injection engineering

Engineering requirements for `epic:adopt-di`.

## What already holds

- Measured on `master` at `d2e3e916e`: the hub's `TenantServices` holds 87 members and its `HubServices` 37; some 220
  route handlers take the whole `TenantServices` through one dependency; four of its members are read by no route.
- The hub's `TenantServices` builder is some 370 lines, in a composition module of more than 1,500 lines. The runner's
  process graph is assembled by hand in `runner/composition.py`, some 1,200 lines.
- `bzh:narrow-seams` already makes each runner loop step declare the slice of `LoopContext` it reads. The hub's API edge
  is exempt from it.
- `bzh:dependency-injection` says blizzard has no dependency-injection container, and records no reason why.
- Pyright runs in strict mode since `epic:strict-types`, and dishka's `provide` is partially unknown to it there.
- The layering gates in `tests/guards/test_layering.py` assume hand wiring: a roster of composition roots, and a check
  that no class is constructed twice across the hub's root files.
- Fifteen of the eighty-two classes in the hub's bags carry constructor defaults.

## Brief requirements

- engineering:pin-dishka-one — blizzard pins dishka's 1.x line and uses only forms not already deprecated, so the move
  to 2.0 is one landing of its own. Serves `epic:adopt-di`.
- engineering:explicit-factory-for-defaults — A constructor with default arguments is registered through an explicit
  factory, since dishka ignores defaults. Serves `epic:adopt-di`.
- engineering:dead-members-deleted — Bag members with no live consumer are deleted rather than registered, after
  checking background and helper callers as well as routes. Serves `epic:adopt-di`.
- engineering:strict-waiver-at-translation — The one pyright strict waiver for dishka is confined to the translation
  modules; every wiring module stays strict-clean. Serves `epic:adopt-di`.

## Container per process (engineering:container-per-process)

The hub, the runner host, and each selected CLI command each build one container, from a composition root, with dishka's
strict validation, so a type claimed twice fails the build. The root's calls into each area's `register` are ordinary
typed calls, so pyright fails a root that omits an adapter an area demands or passes one that does not fit the port.
Serves `product:wiring-beside-code`.

## Area wiring modules (engineering:area-wiring)

Each domain area registers its own services from a wiring module beside its code, through the registry interface in
`foundation/` only. A wiring module aliases each of its ports to the adapter it is handed, declares its own services,
never names a concrete adapter or chooses an adapter's lifetime, and never takes another area's service as an argument.
Adapter packages register each concrete adapter once, with its lifetime, so an adapter serving several areas' ports is
one instance; roots call area registration methods rather than registering individual services. Serves
`product:wiring-beside-code`.

## Born in the container (engineering:born-in-container)

From the landing that activates a process's container path, its new services are constructed by the container and its
legacy builders take no new construction; that landing demonstrates registration and consumption through a production
entrypoint with an exemplar. Serves `product:wiring-beside-code`.

## Narrow seams at the API edge (engineering:narrow-api-edge)

Every route names the services it uses, and every service takes its collaborators as constructor seams rather than a
bag. The hub's API edge falls under `bzh:narrow-seams`. Serves `product:route-names-its-services`.

## Behavior-preserving landings (engineering:behavior-preserving-landings)

Every landing preserves behavior. While a bag and the container both serve a service, the bag's field is filled from the
container, so both hand out the same instance. Serves `product:no-visible-change`.

## Whole graph checked (engineering:graph-checked-whole)

A unit-tier test builds each process's container, and each selected CLI command's, with strict validation and serves no
request, so a missing collaborator or a type with two owners fails the gate. Serves `epic:adopt-di`.

## Every route resolves (engineering:every-route-resolves)

A unit-tier test walks every route and resolves each service it names at the scope that route runs in, since dishka does
not validate what only a route asks for. Serves `product:route-names-its-services`.

## A role is a type (engineering:role-is-a-type)

Where two collaborators share a type, each gets a type of its own, so strict validation refuses a second owner. dishka
resolves by type: a throttle registered the easy way was handed the virtual clock instead of the wall clock, with no
error. Serves `epic:adopt-di`.

## One locked door into a child container (engineering:locked-child-container)

A single helper opens every tenant and runner child container, with its lock, and a gate refuses any other way in.
dishka's child containers take no lock unless given one; without it, thirty-two concurrent first requests built sixteen
stores for one tenant. Serves `epic:adopt-di`.

## One instance on every road (engineering:one-instance-per-port)

Every port an adapter serves resolves to the same object, and while a bag and the container both serve a service they
hand out the same object. A unit-tier gate checks both. Serves `product:no-visible-change` and
`product:wiring-beside-code`.

## dishka stays at the edge (engineering:dishka-at-edge)

Only the translation modules and the route edge import dishka. A domain package imports only the registry interface from
`foundation/`, and no domain class is decorated or marked for the container. `bzh:domain-core` admits the registry
interface and names dishka among the frameworks a domain module may not import. Serves `epic:adopt-di`.

## Wiring modules stay apart (engineering:wiring-modules-apart)

A wiring module imports its own package's types, its ports, and the registry interface, never another area's wiring, and
a `register` parameter is an outward port, never another area's service. A unit-tier gate checks it. Serves
`epic:adopt-di`.

## Ordered teardown (engineering:ordered-teardown)

Lifecycle tests cover failed construction, tenant eviction, runner closure, and process shutdown: owned children close
before shared resources, configured clients close while their owning loop runs, and borrowed inputs are not closed by
the container. Serves `product:no-visible-change`.

## Scope identity and admission (engineering:scope-identity)

The tenant graph is cached across requests and background work; only request-local dependencies are rebuilt per request,
and authentication and tenant admission precede access to that graph. Concurrent first use creates one graph, different
tenants never share tenant-owned services or brokers, and request principals never enter tenant- or process-owned
services. Serves `product:no-visible-change`.

## Selective command imports (engineering:selective-command-imports)

Each selected command loads only its own registration modules, retains invoker-supplied carrier overrides, and closes
only the resources it owns; delayed construction is not evidence of delayed imports. Serves `product:no-visible-change`.

## Rules follow the wiring (engineering:rules-follow-wiring)

`bzh:dependency-injection` and `bzh:narrow-seams` in `blizzard-context` describe which processes have activated the new
path, prohibit new legacy construction there, and follow rolling retirement to the container-only end state with
`decision:adopt-di/container` as rationale. Layering gates and exemplars move with the first usable path and with each
production counterpart's adoption (`bzh:exemplar-conformance`), not ahead of an unavailable mechanism. Serves
`product:wiring-beside-code`.

## Container-only at cutover (engineering:container-only)

As each legacy owner retires, its gates admit only the container; the final gate allows no service bag in a signature
and no owned collaborator constructed outside a registered provider or factory. Typed driver contexts, runtime owners,
plain values, and borrowed thread-confined inputs are not service bags. Serves `product:route-names-its-services`.
