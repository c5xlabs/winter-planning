---
epic: signals
refinement: outline
status: planned
---

# Inbound signals

A standalone epic; its requirements are inline below.

## The problem

Blizzard only ever finds work by being told to look. Everything it does begins with an operator naming an item, which
means the systems that notice trouble first, the error tracker that just saw a new exception or the alerting service
watching a service degrade, have no way to reach the fleet that could act on it. The operator is the wire again, and
this time they are asleep.

## The outcome

Something happens and becomes work: an external system pushes into the fleet, and the item it raises is queued before
morning.

## Depends on

- `initiative:projects`, since an endpoint is configured per project, against a source that project links.

## Requirements

### An inbound endpoint per project (product:project-endpoint)

Each project can be given an inbound endpoint, configured against a source that project links
(`decision:signals/endpoint-owner`).

Acceptance:

- An external system pushes to a project's endpoint, and the item it raises is queued in that project.

### Signed requests (product:signed-pushes)

A push surface on a hub that is reachable from the internet is only safe if the hub can tell who is pushing, so every
push is signed.

Acceptance:

- A push the hub cannot attribute to a configured source is refused.

### Rate bounds per source (product:source-rate-bound)

A flapping alert floods nothing. A source that exceeds its bound is throttled rather than believed.

Acceptance:

- A source pushing faster than its bound is throttled and raises no items past the bound.

### The forge stays the only place work is defined (product:signal-raises-item)

A signal raises an item in a work source the project links, and blizzard ingests it into that project like any other
item, so the forge stays the only place work is defined by construction rather than by discipline.

Acceptance:

- Every item a signal raises appears in the linked work source before blizzard ingests it.

## Open questions

- **Product owner:** What a signal is allowed to say about the work it raises (which graph, which tags, what priority)
  and how much of that a stranger should be trusted with.
- **Product owner:** How a repeating alert is recognized as the same alert, so a service that is down for an hour raises
  one item rather than sixty.
