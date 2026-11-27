---
name: human-answer-forms
external: github:paul-gross/blizzard#2276
depends-on:
  - form-chrome
code:
  - blizzard/web/projects/fleet/src/lib/chunk/chunk-detail/chunk-awaiting-human.ts::ChunkAwaitingHuman
---

# Migrate awaiting-human answers

Delivers [forms.md](../spec/forms.md#interfaces)'s one-form-per-question `AnswerDraft` composition for
`product:full-adoption` and `product:error-at-field`, shared by hub and runner views.

## Change

- Introduce the shared question-form container and field-tree view, with the hub adapting its existing operator-action
  port to the domain saving service. The runner supplies no actions and stays read-only.
- Separate answer fields from the row's action toolbar and preserve the existing answer/decision commands.
- Bind the existing optional decision note through its own domain field tree rather than a private view signal.
- Move shared question data to domain types and remove local answer draft/error/layout helpers.
- Remove the `core/forms` bootstrap exemption and temporary guidance when both app roots reach the shared composition
  that imports the submission helpers (`decision:ui-toolkit/shared-form-bootstrap`), without adding runner mutations.

## Acceptance

- Hub component tests submit independent questions without sharing draft, touched, or pending state.
- Local errors send nothing; hub refusals retain the correct answer/note and focus its control.
- Runner component/browser tests show question and decision trails without editable controls or saving actions; no
  answer/resolve request is sent. Hub permission-denied views likewise remain read-only.
- Action chips remain actions rather than fields; existing resolution mutations keep their semantics.
- `web:typecheck`, `web:unit-test`, and both-app browser cases pass; moved units leave boundary exemptions.
- The placement sweep reports both-app production reach for `core/forms` with no exemption; stories/specs are not used
  to establish that reach.
