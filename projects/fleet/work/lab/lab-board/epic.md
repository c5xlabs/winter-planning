---
epic: lab/lab-board
refinement: outline
status: planned
---

# Lab board

Part of `initiative:lab`; its requirements are inline below.

## The problem

Twenty trials would bury the operator's board, and the operator's board asks the wrong question for them anyway: it asks
what the fleet is doing, where `persona:harness-engineer` is asking what the fleet has learned.

## The outcome

An experiment shows on the operator's board as a single card, and the trials behind it live on a board of their own, the
**lab board**, where an experiment's design, its grid of trials, its running estimate, its budget, and its queue of
blind comparisons are read. It comes after the verdict because it renders what the verdict computes, and before seeded
defects and evidence, which are both read there.

## Requirements

### One card per experiment (product:experiment-card)

An experiment appears on the operator's board as a single card, and none of its trials appear there.

Acceptance:

- The operator's board shows one card per experiment and no trial cards.

### The lab board (product:lab-board-view)

The lab board shows each experiment's design and the overlay each arm carries, a grid of specimens against arms filling
in as trials reach their end states, the running estimate and the uncertainty around it, the budget spent and remaining,
and the queue of blind comparisons waiting for a person's judgement. Any cell in the grid opens the trial as the
ordinary chunk it is, transcripts and all.

Acceptance:

- An engineer reads an experiment's arms and overlays, its grid, its running estimate with uncertainty, and its budget
  spent and remaining.
- An engineer opens any trial from its grid cell, with its transcripts.
- An engineer works through the blind comparisons waiting for them.
