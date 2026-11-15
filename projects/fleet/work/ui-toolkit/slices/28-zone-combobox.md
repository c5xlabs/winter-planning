---
name: zone-combobox
depends-on:
  - cadence-form
  - form-control-bindings
code:
  - blizzard/web/projects/hub/src/app/garden/cadences/cadence-form.ts::FleetCadenceForm
---

# Add and adopt the zone combobox

Delivers `KitCombobox` and `zoneOptions` in [choices.md](../spec/choices.md#interfaces) for `product:picked-values`,
under `decision:ui-toolkit/browser-zones`.

## Change

- Add a form-bound, pick-only combobox over Aria and replace only the cadence zone's free-text control.
- Build sorted browser-zone options plus UTC and offsets at the supplied time.
- Preserve a stored unlisted zone and expose the explicit empty option for an optional zone.

## Acceptance

- Tests pin filtering, UTC inclusion/deduplication, supplied-time offsets, empty selection, and a stored unlisted value.
- Browser tests cover typing, arrow highlight, Enter pick, Escape close, and restoring current text after an unmatched
  edit.
- Hub zone refusals still land on the zone field; `web:typecheck`, `web:unit-test`, and `web:story-render` pass.
