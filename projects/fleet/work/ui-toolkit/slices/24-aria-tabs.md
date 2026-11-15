---
name: aria-tabs
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/kit/kit-tabs.ts::KitTabs
---

# Move in-page tabs to Aria

Delivers [choices.md](../spec/choices.md#interfaces)'s tab/panel composition for `product:keyboard-choices` and
`product:full-adoption`, under `decision:ui-toolkit/angular-aria` and `decision:ui-toolkit/nav-stays-links`.

## Change

- Move `KitTabs` and projected panels to Aria through kit-owned parts; migrate their in-page consumers.
- Preserve active values, labels, content, and styles; routed main-nav `KitTab`/`KitTabStrip` remain links.

## Acceptance

- Browser tests show one Tab entry, Left/Right/Home/End movement, selection following focus, and the matching visible
  panel.
- Tests preserve active-value binding and do not leave the previous hand-wired panel-switch path beside the new one.
- Main-nav links still navigate normally; `web:typecheck`, `web:unit-test`, and `web:story-render` pass.
