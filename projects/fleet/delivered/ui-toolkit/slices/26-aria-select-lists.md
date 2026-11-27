---
name: aria-select-lists
external: github:paul-gross/blizzard#2280
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/kit/kit-select-row.ts::KitSelectRow
---

# Move master lists to Aria

Delivers [choices.md](../spec/choices.md#interfaces)'s `fleetKitSelectList`/`KitSelectRow` composition for
`product:keyboard-choices` and `product:full-adoption`.

## Change

- Replace each column of independently tabbable select rows with a kit-owned vertical Aria listbox.
- Preserve route-backed selection, labels, empty states, and scroll position; distinguish highlighting from opening.

## Acceptance

- A browser case with forty rows has one Tab entry, Up/Down/Home/End/type-ahead movement, and Enter opening the active
  row.
- Route refresh and pointer selection keep the correct active row; tests cover empty and disabled entries.
- `web:typecheck`, `web:unit-test`, and `web:story-render` pass for both standalone kit stories and migrated consumers.
