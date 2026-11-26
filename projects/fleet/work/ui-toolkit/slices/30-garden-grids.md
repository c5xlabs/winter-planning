---
name: garden-grids
external: github:paul-gross/blizzard#2284
depends-on:
  - grid-bridge-members
code:
  - blizzard/web/projects/hub/src/app/garden/cadences/cadence-panel.ts::FleetCadencePanel
  - blizzard/web/projects/hub/src/app/garden/routines/routine-panel.ts::FleetRoutinePanel
---

# Adopt the grid bridge in cadence and routine tables

Completes [choices.md](../spec/choices.md#system)'s three-table adoption for `product:keyboard-choices` and
`product:full-adoption`, under `decision:ui-toolkit/no-table-engine`.

## Change

- The cadence and routine master lists are `fleetKitSelectList` listboxes, not tables, and keep the select list's own
  keyboard model. Their remaining tables (errands, last swept) take no row actions, so under
  `decision:ui-toolkit/no-table-engine` they stay plain rather than joining the grid composition.
- Preserve edit, retire, and run actions, disabled states, and the existing table presentation.

## Acceptance

- Browser tests exercise the select lists' one Tab entry, arrow and type-ahead movement without opening, and Enter and
  click opening exactly once.
- Existing row actions retain their permission/pending behavior and tests; read-only graph/proposal-count tables stay
  plain.
- `web:typecheck`, `web:unit-test`, and the affected browser layout checks pass with no table-engine dependency.
