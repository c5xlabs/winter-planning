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
- engineering:dead-members-deleted — The four `TenantServices` members no route reads are deleted, not carried into the
  container. Serves `epic:adopt-di`.
- engineering:strict-waiver-at-translation — The one pyright strict waiver for dishka is confined to the translation
  modules; every wiring module stays strict-clean. Serves `epic:adopt-di`.

## Container per process (engineering:container-per-process)

The hub, the runner host, and each CLI command group each build one container, from a composition root, with dishka's
strict validation, so a type claimed twice fails the build. The root's calls into each area's `register` are ordinary
typed calls, so pyright fails a root that omits an adapter an area demands or passes one that does not fit the port.
Serves `product:wiring-beside-code`.

## Area wiring modules (engineering:area-wiring)

Each domain area registers its own services from a wiring module beside its code, through the registry interface in
`foundation/` only. A wiring module aliases each of its ports to the adapter it is handed, declares its own services,
never names a concrete adapter or chooses an adapter's lifetime, and never takes another area's service as an argument.
Adapter packages register each concrete adapter once, with its lifetime, so an adapter serving several areas' ports is
one instance. Serves `product:wiring-beside-code`.

## Born in the container (engineering:born-in-container)

From the foundation landing on, a service new to blizzard is constructed by the container; the legacy builders take no
new construction. Serves `product:wiring-beside-code`.

## Narrow seams at the API edge (engineering:narrow-api-edge)

Every route names the services it uses, and every service takes its collaborators as constructor seams rather than a
bag. The hub's API edge falls under `bzh:narrow-seams`. Serves `product:route-names-its-services`.

## Behavior-preserving landings (engineering:behavior-preserving-landings)

Every landing preserves behavior. While a bag and the container both serve a service, the bag's field is filled from the
container, so both hand out the same instance. Serves `product:no-visible-change`.

## Whole graph checked (engineering:graph-checked-whole)

A unit-tier test builds each process's container, and each CLI command group's, with strict validation and serves no
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
hand out the same object. A unit-tier gate checks both. Serves `epic:adopt-di`.

## dishka stays at the edge (engineering:dishka-at-edge)

Only the translation modules and the route edge import dishka. A domain package imports only the registry interface from
`foundation/`, and no domain class is decorated or marked for the container. `bzh:domain-core` admits the registry
interface and names dishka among the frameworks a domain module may not import. Serves `epic:adopt-di`.

## Wiring modules stay apart (engineering:wiring-modules-apart)

A wiring module imports its own package's types, its ports, and the registry interface, never another area's wiring, and
a `register` parameter is an outward port, never another area's service. A unit-tier gate checks it. Serves
`epic:adopt-di`.

## Ordered teardown (engineering:ordered-teardown)

A shutdown test shows each tenant's connections closing before the engine they share is disposed. Serves
`epic:adopt-di`.

## Rules follow the wiring (engineering:rules-follow-wiring)

`bzh:dependency-injection` and `bzh:narrow-seams` in `blizzard-context` change twice. At the foundation they describe
the transition: the container is each process's composition graph, new services are born in it, and the builders and
bags are a path being retired that admits nothing new. At cutover they describe the end state, with the reasoning in
`decision:adopt-di/container`. The layering gates are restated against the translation and wiring modules at the
foundation. The exemplars move with the code they model (`bzh:exemplar-conformance`): the exemplar wiring module at the
foundation, and each exemplar service the day its production counterpart adopts. Serves `product:wiring-beside-code`.

## Container-only at cutover (engineering:container-only)

At cutover the gates that admitted both paths admit only the container: no bag type appears in a signature, and no
process collaborator is constructed outside a provider. Serves `product:route-names-its-services`.
