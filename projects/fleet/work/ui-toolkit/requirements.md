# UI toolkit requirements

Requirements of `epic:ui-toolkit`.

## Mistakes are caught as they are made (product:immediate-validation)

A form tells the operator what is wrong with a field as they fill it in, before anything is sent: a required field left
empty, or a value that is not the shape the field takes. The form marks its required fields before the operator types,
and Save sends nothing while a field is in error (`decision:ui-toolkit/validate-both`).

Acceptance:

- Every required field carries a required marker before anything is typed, and no optional field carries one.
- A field in error shows its error once the operator leaves it, and every field in error shows its error on Save.
- Save on a form with a field in error sends no request and moves focus to the first field in error.
- The repository and work-source forms require a forge (`decision:ui-toolkit/forge-required`).

## A refusal lands on its field (product:error-at-field)

The hub still judges every save. When it refuses one the frontend let through, the operator sees the refusal beside the
field it concerns, and that field takes focus. A refusal that concerns no field shows under the form.

Acceptance:

- A refusal that names a field, by its path or by its kind (a slug already taken), shows under that field, and focus
  moves to it.
- A refusal that concerns no field shows under the form, as it does today.
- The typed draft survives the refusal.

## A typo never changes a stored value (product:no-silent-reset)

A field that takes a number accepts only a number, and a field whose blank means "the default" makes the default a
choice the operator makes (`decision:ui-toolkit/explicit-default`). Nothing the operator mistypes is saved as something
else.

Acceptance:

- Text in a number field that is not a number is that field's error, and nothing is sent.
- A park timeout returns to its default only when the operator chooses the default.
- A blank number field is either a required-field error or the default the operator chose, never a silent reset.

## Every choice works from the keyboard (product:keyboard-choices)

Tab strips, single-choice chip rows, row lists, accordion headers, and the rows of a table whose rows act are each one
Tab stop, moved through with the arrow keys, following the WAI-ARIA pattern each one is.

Acceptance:

- In a tab strip, the arrow keys move between tabs and Home and End reach the ends.
- A single-choice chip row is announced as one group with one chosen value, and the arrow keys change the choice.
- A master list is one Tab stop, the arrow keys move the highlighted row, and Enter opens it.
- Accordion headers are reached with the arrow keys.
- A table whose rows act is one Tab stop, and its cells are reached with the arrow keys.

## A zone picked, not remembered (product:picked-values)

A cadence's zone is picked from the zones there are, filtered as the operator types, rather than typed from memory
(`decision:ui-toolkit/browser-zones`).

Acceptance:

- A cadence's zone is picked from the IANA zone list, filtered by what is typed.
- A zone that is not on the list cannot be entered, and an optional zone can be cleared.
- A cadence whose stored zone is not on the list shows that zone and keeps it until the operator picks another.

## The frontend owns its domain (product:domain-boundary)

An agent working on the board works in the board's own domain types, and the wire is translated at one boundary
(`decision:ui-toolkit/frontend-domain`). A wire change is absorbed in the translator for that concept, and a component
never reads a generated wire type.

Acceptance:

- blizzard-context carries `bzh:frontend-domain-boundary` in place of `bzh:frontend-wire-conformist`, and no rule
  there still cites the old one.
- No file outside the data-access files and the translators imports a generated wire type, and the structural gate
  fails one that does.
- The boundary's exemption list is empty.
- An exemplar shows a concept's domain model, translator, and service, so the next agent to add one copies it.

## Every form and choice moves (product:full-adoption)

The epic ends with one way to build a form and one keyboard model per choice, not two of each side by side. Every form
the board has moves to Signal Forms, its domain model, and the kit's field part. Every kit part whose keyboard model
Angular Aria supplies moves to it. No surface is left on the old way.

Acceptance:

- All eleven form surfaces, including the admin config dialog and all five record types it serves, are built on Signal
  Forms, a domain model, and `KitField`. No form keeps hand-written set and save helpers, a field-layout class, or an
  error line of its own.
- `KitTabs`, `KitChips` and the chip rows built from single `KitChip`s, the `KitSelectRow` lists, and
  `KitAccordionSection` are on Angular Aria, as are the members, cadences, and routines tables.
- The superseded code is deleted, not left beside its replacement: the per-form layout classes, the required and
  number checks in the admin field lists, `missingRequired`, `invalidNumbers`, and the text-to-number conversion in
  `config-edit.model.ts`.

## A path for the next surface (product:assembled-surface)

After adoption, the way to add a form is to assemble one, demonstrated by building the next real surface the board needs
from parts.

Acceptance:

- The next new board form after adoption is built from a domain model, its service, Signal Forms, and the kit's field
  part, with no draft, layout, or error handling of its own.
- An exemplar shows that path, so the next agent to add a form copies it.
