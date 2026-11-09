---
name: adoption
depends-on:
  - foundation
---

# Adoption

Moves each service's construction from the legacy builder into its area's wiring module, unchanged in every other
respect, as a series of landings whose size waits on `decision:adopt-di/adoption-batch-size`. Delivers
`engineering:area-wiring`, `engineering:explicit-factory-for-defaults`, and `engineering:behavior-preserving-landings`;
shaped by `decision:adopt-di/strangler-migration`.

## Change

- A service adopts once every collaborator it takes is already in the container: the stores and the process's core
  first, then the services that take only stores, then the services above them.
- The hub adopts first, then the runner host and the CLI; within a process the batch is a concept package's services,
  leaves first.
- An adopted service keeps its constructor signature, and every service that took the bag still takes it.
- The bag's field for an adopted service is filled from the container.
- A constructor with default arguments gets an explicit factory.
- Each exemplar service moves the day its production counterpart adopts.

## Acceptance

- Once adoption ends, the builders construct nothing and only assemble bags from what the container hands them.
- The bag and the container hand out the same instance of every adopted service.
- Behavior is unchanged at every route and command.
