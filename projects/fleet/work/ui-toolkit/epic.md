---
epic: ui-toolkit
refinement: refined
status: planned
---

# UI toolkit

A standalone epic. Its product requirements are in `requirements.md`.

## The problem

The board already builds its chrome from a shared kit, which the lint gate enforces. What the kit does not cover is
still built by hand on each surface, and two kinds of surface show it most.

The first is forms. Each of the board's eleven forms keeps its own draft, lays out its own labels, and decides on its
own which fields are required. Little is checked before the hub refuses the save, and then the operator gets one red
sentence under the whole form and has to find the field it means. A number field with a typo greys out Save without
saying why, and clearing a park timeout quietly resets it to its default.

The second is choices. Tab strips, choice chips, row lists, and accordion headers have no keyboard model: a list of
forty rows is forty Tab stops, and a single choice among four chips reads as four unrelated toggles. A timezone is typed
from memory and judged only when the save fails.

Beneath both, the board has no domain of its own. Every component, model, and form works in the generated wire types,
about two hundred files outside the data-access layer, because the frontend's conventions forbid it from modelling
anything the wire carries. A form cannot hold a rule the wire does not state, and a wire change reaches every file that
read the field.

The cost of writing each of these is the visible half. The expensive half is drift: forms written six months apart lay
out and report errors in different ways, and the operator learns each of them separately.

## The outcome

The board models its own domain and translates the wire at one boundary, as the backend does
(`decision:ui-toolkit/frontend-domain`). Every form is assembled one way: a domain model, a service that saves it,
Angular Signal Forms (`decision:ui-toolkit/signal-forms`), and a kit field part that shows the label, whether the field
is required, a hint, and the field's own error. A form catches a mistake as the operator makes it, and the hub still
judges every save, with its refusals landing on their fields (`decision:ui-toolkit/validate-both`). The kit's choice and
navigation parts take their keyboard model from Angular Aria (`decision:ui-toolkit/angular-aria`), and the kit gains the
combobox the board has lacked.

Both adoptions follow the move to Angular 22, where both libraries are stable (`decision:ui-toolkit/angular-22-first`).
The landscape this epic starts from and the candidates it weighed are in [research/landscape/](./research/landscape/).
What each change looks like, before and after, is in [assets/](./assets/), and the order it lands in is
`decision:ui-toolkit/landing-order`.

## What this epic is not

- **Not a redesign.** The board keeps its visual language. The only visible changes are states it does not have today:
  a field's own error, a required marker, a focus ring that moves with the arrow keys, and an open picker.
- **Not a wire change.** No route, request, response, or schema changes on the hub or the runner
  (`decision:ui-toolkit/no-wire-change`). Where the form is stricter than the wire, as with a repository's forge, the
  form holds the rule (`decision:ui-toolkit/forge-required`).
- **Not a wholesale replacement of the kit.** Only the forms and the choice parts move (`product:full-adoption`).
  `KitMenu` and `KitSelect`, the read-only tables, the main-nav tabs, and the rest of the kit stay as they are
  (`decision:ui-toolkit/cdk-parts-stay`, `decision:ui-toolkit/no-table-engine`, `decision:ui-toolkit/nav-stays-links`),
  and no styled component suite is adopted (`decision:ui-toolkit/no-component-suite`).
- **Not more pickers.** Only the cadence zone becomes a picker. Other fields that could pick from a list the hub serves,
  such as a token secret, a routine, a scope, a graph, or an absorbing finding, stay as they are.

## Depends on

- `epic:storybook`, delivered, whose workshop is where each new and reshaped part is judged.
