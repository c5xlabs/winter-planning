---
name: aria-chips-toolbars
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/kit/kit-chips.ts::KitChips
  - blizzard/web/projects/fleet/src/lib/kit/kit-chips.ts::KitChip
---

# Separate single-choice chips from action toolbars

Delivers [choices.md](../spec/choices.md#system)'s listbox and toolbar compositions for `product:keyboard-choices` and
`product:full-adoption`, under `decision:ui-toolkit/angular-aria`.

## Change

- Move single-choice chip groups, including projected individual chips, to one Aria selection model.
- Move action-chip rows to kit-owned toolbar directives, not a selection field.
- Migrate project lenses, filters, colors, graph rows, and awaiting-human action rows without changing their actions.

## Acceptance

- Browser tests distinguish one selected option/group from an action toolbar, with one Tab entry and arrow navigation.
- Enter/Space runs each action exactly once; disabled chips do not run it.
- Existing value bindings and action outputs remain correct in both apps; `web:typecheck`, `web:unit-test`, and
  `web:story-render` pass.
