---
name: form-chrome
external: github:paul-gross/blizzard#2259
depends-on:
  - form-save-contracts
  - form-control-bindings
---

# Add shared field and form chrome

Builds `KitForm`, `KitField`, `KitNumberInput`, and `KitOrDefault` from [forms.md](../spec/forms.md#interfaces), serving
`product:immediate-validation`, `product:error-at-field`, and `product:no-silent-reset`.

## Change

- Render labels, required markers, hints, touched errors, and root refusals from field state.
- Keep numeric text as typed and make default-versus-stated-value an explicit bound choice.
- Centralize the action row and submitting state without adopting an application form yet.

## Acceptance

- Component tests distinguish required from optional fields before typing, touched from untouched errors, and
  field-specific from root refusals.
- The default toggle disables its projected control without losing typed text; clearing a stated number remains invalid.
- Browser checks demonstrate focus movement through the bound native controls and usable narrow-width errors.
- All new parts have stories; `web:typecheck`, `web:unit-test`, and `web:story-render` pass.
