---
epic: adopt-di
refinement: refined
status: planned
---

# Adopt dependency injection

A standalone epic; its requirements are inline below.

## The problem

Most of blizzard is written by agents meeting a module for the first time (`persona:fleet-agent`), and an agent adding a
service to the hub learns how it is wired by editing the wiring. It adds a field to a frozen bag of services, adds a
constructor call to a builder some 370 lines long, and does both inside a composition module of more than 1,500 lines
that every other agent adding a service is editing the same week. The route that uses the new service then takes the
whole bag, so its signature says nothing about what it touches, and a reviewer reading it cannot tell which state a
change can reach.

The bags have grown faster than anyone was watching. Where in mid-July a single bag held 11 services, the hub's two bags
now hold well over a hundred between them, and some 220 route handlers take the whole tenant bag. In a flat list that
long, nobody notices when a service stops being used. A bag that every caller reaches into is a service locator by
another name: it makes depending on everything the cheapest dependency to write. The runner's process graph is assembled
by hand in the same way.

## The outcome

The hand-built graphs of the hub, the runner host, and the CLI that drives them are replaced by a dependency-injection
container (`decision:adopt-di/container`): a registry that knows how to build each service from the services it needs,
builds each one once for the lifetime it is declared with, hands a route exactly the services that route asks for, and
tears everything down in the reverse of the order it was built.

An agent adding a service wires it beside its own code, in its own area, instead of in the module every other agent is
editing. A route names the services it uses, so a reviewer reads what a change can reach from its signature. And the
change happens the way a strangler fig replaces a tree (`decision:adopt-di/strangler-migration`): the new structure
grows around the old one, takes over its load one branch at a time, and the old one is cut away only when nothing rests
on it, so at no point does the hub or the runner stop working while it happens. The epic is delivered when the hub, the
runner host, and the CLI are wired only through their containers.

## What this epic is not

The web board, the mock fleet in `blizzard-mock`, and any change to what blizzard does are untouched. Splitting the bags
into smaller bags is not done (`decision:adopt-di/narrow-not-split`).

## Requirements

### Wiring beside the code (product:wiring-beside-code)

An agent adding a service declares how it is built in its own area's wiring, beside its code, and never edits a shared
composition module to do it. From the first landing of this epic, a service new to blizzard is born this way.

Acceptance:

- An agent adds a service to the hub, the runner host, or the CLI without editing another area's code or a shared
  builder.
- An exemplar shows the new path, so the first agent to add a service imitates it.

### A route names what it uses (product:route-names-its-services)

A route, and a service that reached into a bag, names the services it uses rather than taking the bag, so a reviewer can
tell from its signature which state a change can reach.

Acceptance:

- No route or service takes a bag of services once the epic is delivered.

### Nothing visible changes (product:no-visible-change)

Wiring is not a released surface. Nothing an operator, a runner on a supported release, or a client of the API can see
changes: the HTTP routes, the CLI, the configuration, and the wire models are untouched, and every landing preserves
behavior, so removing the old path is the epic's own last step rather than a follow-up break.

Acceptance:

- Every route, command, configuration, and wire model behaves as before each landing.
