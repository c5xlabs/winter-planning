---
name: foundation
---

# Foundation

Gives the hub, the runner host, and the CLI each a container and a composition root holding today's graph whole, and
lands every guard the later slices lean on. Delivers `engineering:container-per-process`,
`engineering:born-in-container`, `engineering:dead-members-deleted`, the guard requirements, and the transitional form
of `engineering:rules-follow-wiring`; spec `System` and `Domain`. Shaped by `decision:adopt-di/strangler-migration`.

## Change

- Each process gains its container, built with strict validation, and its composition root.
- A legacy module per process answers for today's graph by asking the existing builders for the instances they already
  make: the hub's for `HubServices`, `TenantServices`, and the stores beneath them; the runner host's for its host
  stores, each runner's process collaborators, and each runner's `LoopContext`.
- A tenant is a child container held in the hub's existing tenant cache, and each runner a child container under its
  host, opened through the one locked helper.
- The registry interface lands in `foundation/`, with one translation module per process, and an exemplar area wiring
  module beside the root call that hands it its adapters.
- The guards land as unit-tier gates: `engineering:graph-checked-whole`, `engineering:every-route-resolves`,
  `engineering:role-is-a-type`, `engineering:locked-child-container`, `engineering:one-instance-per-port`,
  `engineering:dishka-at-edge`, `engineering:wiring-modules-apart`, `engineering:ordered-teardown`.
- dishka is pinned to its 1.x line, with the strict waiver confined to the translation modules.
- `bzh:dependency-injection` and `bzh:narrow-seams` take their transitional form; `bzh:domain-core` admits the registry
  interface and names dishka as forbidden in domain modules; the layering gates are restated against the translation and
  wiring modules.
- The four `TenantServices` members no route reads are deleted.
- Leaves alone what every route and loop step receives, and who builds every existing service.

## Acceptance

- Each process's container builds under strict validation and every route resolves.
- Every guard gate passes on the tree and fails on a planted violation.
- The shutdown test shows each tenant's connections closing before the shared engine is disposed.
- Behavior is unchanged at every route and command.
