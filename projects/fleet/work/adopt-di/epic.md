---
epic: adopt-di
refinement: final
status: ready
external: github:paul-gross/blizzard#2352
---

# Adopt dependency injection

A standalone epic; its requirements are inline below.

## The problem

Most of blizzard is written by agents meeting a module for the first time (`persona:fleet-agent`), and an agent adding a
service to the hub learns how it is wired by editing the wiring. It adds a field to a frozen bag of services, adds a
constructor call to a builder some 630 lines long, and does both inside a composition module of some 2,200 lines that
every other agent adding a service is editing the same week. The route that uses the new service then takes the whole
bag, so its signature says nothing about what it touches, and a reviewer reading it cannot tell which state a change can
reach.

The bags have grown faster than anyone was watching. Where in mid-July a single bag held 11 services, the hub's two bags
now hold some 160 between them, and some 260 route handlers take the whole tenant bag. In a flat list that long, nobody
notices when a service stops being used. A bag that every caller reaches into is a service locator by another name: it
makes depending on everything the cheapest dependency to write. The runner's process graph is assembled by hand in the
same way.

## The outcome

The hand-built graphs of the hub, the runner host, and the CLI that drives them are replaced by a dependency-injection
container (`decision:adopt-di/container`): a registry that knows how to build each service from the services it needs,
builds each one once for the lifetime it is declared with, hands a route exactly the services that route asks for, and
tears everything down in the reverse of the order it was built.

An agent adding a service wires it beside its own code, in its own area, instead of adding another service to a central
builder. The process root composes areas and chooses their adapters; each area owns the registration of its services. A
route names the services it uses, so a reviewer reads what a change can reach from its signature. And the change happens
the way a strangler fig replaces a tree (`decision:adopt-di/strangler-migration`): the new structure grows around the
old one, takes over its load one branch at a time, and the old one is cut away only when nothing rests on it, so at no
point does the hub or the runner stop working while it happens.

## What this epic is not

The web board, the mock fleet in `blizzard-mock`, and any change to what blizzard does are untouched. Splitting the bags
into smaller bags is not done (`decision:adopt-di/narrow-not-split`).

## Requirements

### Wiring beside the code (product:wiring-beside-code)

An agent adding a service to an existing area declares how it is built in that area's wiring, beside its code, rather
than in a shared builder. A root changes to introduce an area or choose its adapter bindings, not to maintain a list of
individual services; each process supplies a working example before new services there must follow the new path.

Acceptance:

- An agent adds a service within an already composed area of the hub, runner host, or CLI without editing the process
  root or another area's wiring, provided the area's adapter bindings do not change.
- Introducing an area or changing its adapter bindings changes the root's area-level calls, not a central service list.
- Each process's first usable container path has an exemplar showing how to register and consume a new service.

### A route names what it uses (product:route-names-its-services)

A route, and a service that reached into a bag, names the services it uses rather than taking the bag, so a reviewer can
tell from its signature which state a change can reach.

Acceptance:

- No route or service takes a bag of services once the epic is delivered; the hub, runner host, and CLI obtain their
  owned collaborators through their containers.

### Nothing visible changes (product:no-visible-change)

Wiring is not a released surface. Nothing an operator, a runner on a supported release, or a client of the API can see
changes: the HTTP routes, the CLI, the configuration, and the wire models are untouched, and every landing preserves
behavior, so removing the old path is the epic's own last step rather than a follow-up break.

Acceptance:

- Every route, command, configuration, and wire model behaves as before each landing, apart from the two responses
  `decision:adopt-di/drained-runner-request` and `decision:adopt-di/closed-tenant-request` change for a request racing
  its runner's retirement or its tenant's deletion.
