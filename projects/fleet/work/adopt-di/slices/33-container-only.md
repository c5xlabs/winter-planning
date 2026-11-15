---
name: container-only
depends-on:
  - hub-legacy-retirement
  - runner-api-retirement
  - cli-hub-local
  - cli-runner-local
  - cli-offline-invariants
---

# Container-only closure

Closes spec [Mixed ownership](../spec/migration.md#flows), delivering `engineering:container-only`,
`engineering:rules-follow-wiring`, and the end state of `product:route-names-its-services`. Follows
`decision:adopt-di/declared-cutover`; this is a final audit/enforcement change, not a deferred cleanup phase.

## Change

- Remove remaining empty transition plumbing and migration-only inventory allowances.
- Enforce area-level roots, registered collaborator ownership, no service-bag signatures, one adapter instance per port
  family, and the single locked child opener across hub, host and selected command graphs.
- Put harness guidance and all affected exemplars in their final state, retaining the explicit distinction between
  service bags, lifecycle owners, typed driver contexts, borrowed values and plain records.

## Acceptance

- Full graph/route/command inventory is covered; planted bag parameters, root service registrations, duplicate
  ownership, raw child opening and unregistered owned construction each fail their gate.
- No legacy provider/bag/assembler remains; typed loop contexts and bounded lifecycle owners remain valid.
- `blizzard:typecheck`, structural gates, relevant unit/component/service tests, CLI/config/SSE/compatibility contracts
  and lifecycle/tenant/runner isolation tests pass without released-surface changes.
- Harness/exemplar conformance checks pass; adding a service in an existing area changes only its wiring and consumers,
  while adding an area changes the root's area call.
