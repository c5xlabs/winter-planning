---
name: form-control-bindings
depends-on:
  - domain-boundary-gate
code:
  - blizzard/web/projects/fleet/src/lib/kit/kit-text-input.ts::KitTextInput
  - blizzard/web/projects/fleet/src/lib/kit/kit-checkbox.ts::KitCheckbox
  - blizzard/web/projects/fleet/src/lib/kit/kit-select.ts::KitSelect
  - blizzard/web/projects/fleet/src/lib/kit/kit-color-input.ts::KitColorInput
  - blizzard/web/projects/fleet/src/lib/kit/kit-option.ts::KitOption
---

# Bind kit controls to Signal Forms

Builds [forms.md](../spec/forms.md#system)'s `FormValueControl` integrations and native-radio option group, preparing
`product:full-adoption` under `decision:ui-toolkit/signal-forms` and `decision:ui-toolkit/kit-dependencies`.

## Change

- Bind value, disabled, touched, and focus to the actual native controls, including multiline text and radio options.
- Add the option-group composition while retaining existing non-form control inputs and outputs.
- Keep the select's CDK listbox/overlay and all standalone control behavior.

## Acceptance

- Component tests bind each control through `[formField]` and exercise value, touched, disabled, and focus propagation.
- A real-browser radio case has one native group, arrow selection, and correct form value; multiline text focuses the
  textarea rather than its wrapper.
- `web:typecheck`, `web:unit-test`, and `web:story-render` pass with both legacy-use and form-bound stories.
