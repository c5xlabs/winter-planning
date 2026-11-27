---
name: aria-accordions
external: github:paul-gross/blizzard#2281
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/kit/kit-accordion-section.ts::KitAccordionSection
---

# Move accordion groups to Aria

Delivers [choices.md](../spec/choices.md#system)'s group/trigger/panel composition for `product:keyboard-choices` and
`product:full-adoption`.

## Change

- Put accordion keyboard/expanded behavior behind the kit group directive and section component.
- Migrate existing accordion consumers while preserving their content, expansion bindings, and visual language.

## Acceptance

- Browser tests exercise Up/Down across headers and Enter/Space expansion with correct trigger/panel relationships.
- Disabled and initially expanded states remain correct; no duplicate native/wrapper Tab stops are introduced.
- `web:typecheck`, `web:unit-test`, and `web:story-render` pass.
