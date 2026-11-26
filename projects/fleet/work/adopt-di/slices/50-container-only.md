---
name: container-only
depends-on:
  - hub-legacy-retirement
  - runner-api-retirement
  - cli-hub-local
  - cli-runner-local
  - cli-runner-client
  - cli-worker
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
  service bags, lifecycle owners, typed driver contexts, borrowed values and plain records. `bzh:dependency-injection`
  states the container-only end state and cites `decision:adopt-di/container`.

## Acceptance

- `blizzard:structural-gate`: the inventory gate has no migration allowance left; a planted bag parameter and raw child
  opening each fail it.
- `blizzard:unit-test` (the registry guards in `tests/guards`, over each process's recorded registry): a planted root
  service registration, duplicate port owner and unregistered owned construction each fail them.
- `blizzard:unit-test` (the same guards): every parameter type of a registered constructor, factory or route handler is
  a registered key, a context value, or a type the spec's end-state table classifies; a planted unclassified type fails.
- `blizzard:structural-gate`: no legacy provider/bag/assembler remains, and a planted one fails; typed loop contexts and
  bounded lifecycle owners remain valid.
- `blizzard:typecheck`, `blizzard:unit-test`, `blizzard:component-test` and `blizzard:service-test` pass, with
  `blizzard:cli-contract`, `blizzard:config-contract`, `blizzard:sse-contract` and `blizzard:compat` unchanged,
  including lifecycle and tenant/runner isolation tests.
- `blizzard-context:reference-lint` and `blizzard-context:registry-drift` pass, and `bzh:dependency-injection` names the
  container-only end state; the final exemplars pass `blizzard:typecheck`, `blizzard:unit-test` and
  `blizzard:component-test`.
- `blizzard:unit-test` (the same guards): a planted service added in an existing area passes with edits only to its area
  wiring and consumers, and a planted root service registration fails.
