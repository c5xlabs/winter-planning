---
name: domain-boundary-gate
external: github:paul-gross/blizzard#2256
depends-on:
  - angular-22
code:
  - blizzard/web/scripts/wire-conformist-sweep.js::assertClientCallPlacementDetectorWorks
  - blizzard/web/scripts/generate-stub-operations.mjs::table
  - blizzard/web/projects/fleet/src/lib/testing/stub-request-client.ts::stubAnswerRaceLost
---

# Establish the domain boundary and its gate

Delivers [boundary.md](../spec/boundary.md#system)'s rule replacement, import detector, and migration exemptions for
`product:domain-boundary`, under `decision:ui-toolkit/frontend-domain` and
`decision:ui-toolkit/incremental-replacement`.

## Change

- Replace the conformist rule and its citations in blizzard-context, including the domain, form, and service placement
  and dependency rules; narrow open-vocabulary handling to translators.
- Add the symbol-origin boundary detector and seed its exemption list from the actual current tree, not the survey.
- Retain client-call placement and backend-citation checks; update the verification guidance with the new detector.
- Establish [boundary.md](../spec/boundary.md#system)'s test-only transport home: move the typed stub, generated
  operation tables, refusal constructors, and their focused specs there; update generation and retain `fleet/testing`
  exports.
- Move wire builders and auth-response fixtures to pure translator fixture modules without changing their HTTP shapes or
  requiring presentation consumers to migrate early.

## Acceptance

- Detector self-tests reject direct, aliased, namespace, `import()`-type, and transitive barrel imports of generated
  types, including `fleet`; same-named domain declarations and permitted boundary imports pass.
- `web:structural-gate` passes with only the enumerated migration exemptions, and rejects a planted unexempted consumer.
- Lint probes reject Angular in domain/translators, non-Signal-Forms Angular imports in form rules, and Aria outside the
  kit. The updated context contains no live citation of the retired conformist rule.
- Fixtures permit wire imports in the bounded test transport and translator fixtures, reject them in ordinary test
  helpers and component/model specs, and reject production/story reach into transport through direct or indirect
  imports.
- Regenerating the clients and operation tables leaves no diff after staging; typed stub tests still check paths,
  responses, and refusals. `web:typecheck`, `web:unit-test`, and `web:story-render` pass with the existing public
  helpers.
- Test transport and wire fixture modules need no `DOMAIN_BOUNDARY_EXEMPT` entry; unmoved presentation consumers retain
  only their enumerated migration exemptions.
