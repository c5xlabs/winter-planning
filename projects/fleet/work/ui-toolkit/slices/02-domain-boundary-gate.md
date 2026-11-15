---
name: domain-boundary-gate
depends-on:
  - angular-22
code:
  - blizzard/web/scripts/wire-conformist-sweep.js::assertClientCallPlacementDetectorWorks
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

## Acceptance

- Detector self-tests reject direct, aliased, namespace, `import()`-type, and transitive barrel imports of generated
  types, including `fleet`; same-named domain declarations and permitted boundary imports pass.
- `web:structural-gate` passes with only the enumerated migration exemptions, and rejects a planted unexempted consumer.
- Lint probes reject Angular in domain/translators, non-Signal-Forms Angular imports in form rules, and Aria outside the
  kit. The updated context contains no live citation of the retired conformist rule.
