---
name: retire-boundary-compatibility
depends-on:
  - retire-config-legacy
  - chore-forms
  - finding-triage-form
  - proposal-pass-form
  - run-form
  - routine-forms
  - scope-forms
  - human-answer-forms
  - new-item-form
  - shared-core-domains
  - shared-chunk-domains
  - board-domains
  - delivery-domains
  - hub-read-domains
  - runner-host-domains
  - shell-auth-domains
  - admin-history-domains
  - member-domains
code:
  - blizzard/web/scripts/wire-conformist-sweep.js::assertClientCallPlacementDetectorWorks
---

# Retire the wire-projection compatibility paths

Closes [boundary.md](../spec/boundary.md#system)'s migration under `decision:ui-toolkit/incremental-replacement`,
delivering the empty-exemption acceptance of `product:domain-boundary`.

## Change

- Delete old query projections, compatibility exports, and obsolete wire builders whose consumers have moved.
- Empty the boundary exemption list; keep generated clients, permitted focused boundary tests, and their wire fixtures.
- Reconcile the implemented gate's guidance and examples with the final domain boundary.

## Acceptance

- `DOMAIN_BOUNDARY_EXEMPT` is empty and `web:structural-gate` passes without ignoring component/model specs or stories.
- Detector fixtures still reject indirect generated imports through `fleet` and nested barrels after compatibility
  removal.
- `web:typecheck`, `web:unit-test`, `web:story-render`, and `blizzard:exemplar-check` pass.
- A source census finds no obsolete query projection or live citation of the retired conformist rule.
- The census includes admin changes/history/counts and members. Typed transport and wire fixtures pass in their
  specified test-only homes with no boundary exemption; regeneration cannot recreate a forbidden helper import.
- The temporary `core/forms` placement exemption is absent; its both-app reach still passes the existing placement gate.
