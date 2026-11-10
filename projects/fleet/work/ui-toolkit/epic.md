---
epic: ui-toolkit
refinement: outline
status: planned
---

# UI toolkit

A standalone epic; its requirements are inline below.

## The problem

Every small surface the board grows, a form, a picker, a confirmation sheet, is invented, styled, and maintained by
hand, and the count climbs with every epic that touches it. The cost of writing each one is the visible half. The
expensive half is drift: two dialogs written six months apart agree on nothing, and the operator learns each of them
separately.

## The outcome

The next surface is assembled from a toolkit, which is faster to write and quieter to use. In the spirit of
`initiative:config`, this is a survey first and an adoption second.

## What this epic is not

Wholesale replacement of the hand-built pieces is not the goal and would not pay.

## Depends on

- `epic:storybook`, whose workshop is where the survey's candidates are judged.

## Requirements

### The survey (product:toolkit-survey)

A deliberate look at the component libraries and frontend accelerants the web surface does not yet use, judged against
the board's real needs: the forms, tables, sheets, and pickers it already has and the ones its roadmap will ask for.

Acceptance:

- The survey names the libraries it looked at and judges each one against the forms, tables, sheets, and pickers the
  board has and its roadmap asks for.

### One choice, made on evidence (product:toolkit-choice)

The candidate that wins does so on a demonstration rather than a feature matrix, and the reasoning is written down where
the next person to ask can find it.

Acceptance:

- The choice rests on a demonstration, and its reasoning is written down where the next person can find it.

### Migration where the payoff is real (product:payoff-migration)

Hand-built pieces move to the toolkit when the move buys something; the rest stay.

Acceptance:

- Each hand-built piece that moved to the toolkit has a stated reason it was worth moving.
- Hand-built pieces with no such reason are still in place.

### A path for the next surface (product:assembled-surface)

After adoption, the way to add a form is to assemble one, demonstrated by building the next real surface the board needs
from parts.

Acceptance:

- The next new board surface after adoption is built from the toolkit's parts.

## Open questions

- **Engineering:** Whether a choice worth making constrains the existing frontend stack, and what an honest answer costs
  if it does.
- **Product owner:** What counts as evidence: the most convincing experiment is building one real board surface twice,
  which is also the most expensive.
- **Product owner:** How the epics already queued against the board (the story map, `epic:intake`, and
  `epic:self-sourced-work`'s authoring surface) sequence around this one, since each is cheaper after it and blocked
  waiting for it.
