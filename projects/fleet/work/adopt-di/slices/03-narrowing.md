---
name: narrowing
depends-on:
  - adoption
---

# Narrowing

Teaches each consumer of a bag to ask for what it uses, one consumer at a time. Delivers `engineering:narrow-api-edge`
and `product:route-names-its-services`; shaped by `decision:adopt-di/narrow-not-split`.

## Change

- A route that read services off `TenantServices` names those services.
- A service that took a bag to reach into it takes its collaborators as constructor seams.
- The hub's API edge falls under `bzh:narrow-seams`.
- Leaves the bags whole; none is split into sub-bags.

## Acceptance

- Each narrowed route and service names only what it uses, and every route still resolves.
- Behavior is unchanged at every route and command.
