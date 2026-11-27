---
name: hub-read-domains
external: github:paul-gross/blizzard#2289
depends-on:
  - shared-core-domains
  - proposal-pass-form
  - finding-triage-form
  - routine-forms
  - scope-forms
  - run-form
code:
  - blizzard/web/projects/hub/src/app/graphs/graphs-page.ts::GraphsPage
  - blizzard/web/projects/hub/src/app/events/event-detail-page.ts::EventDetailPage
  - blizzard/web/projects/hub/src/app/garden/gardening-page.ts::GardeningPage
---

# Complete the hub's read-only feature domains

Moves the graph/event pages and remaining garden overview/detail/count readers through
[boundary.md](../spec/boundary.md#system), serving `product:domain-boundary`.

## Change

- Reuse garden domains already introduced with forms; add read translators for the remaining graph/event concepts.
- Move the existing proposal-acceptance view's data inputs onto domains without rebuilding its form before the capstone.
- Preserve navigation, graph selection, event streaming, and all three read-only tables.

## Acceptance

- Translator/model tests cover unknown vocabularies and empty/failed reads while preserving existing routes and filters.
- The old proposal-acceptance form still submits correctly with domain inputs.
- `web:typecheck`, `web:unit-test`, and affected browser layout cases pass; migrated read units leave exemptions.
- No read-only table acquires grid interaction, sorting, or a table-engine dependency.
