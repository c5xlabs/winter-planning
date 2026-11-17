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

- Put cadence and routine interactive tables on the shared grid composition without adding sorting or filtering.
- Preserve edit, retire, and run actions, disabled states, and the existing table presentation.

## Acceptance

- Browser tests exercise one Tab entry, text/action-cell navigation, native focus, and exactly-once activation in each
  table.
- Existing row actions retain their permission/pending behavior and tests; read-only graph/proposal-count tables stay
  plain.
- `web:typecheck`, `web:unit-test`, and the affected browser layout checks pass with no table-engine dependency.
