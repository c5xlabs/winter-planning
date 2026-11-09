---
epic: tagging
refinement: outline
status: planned
---

# Work tagging

A standalone epic; its requirements are inline below.

## The problem

An operator with three machines has three identical appetites, because a runner has no way to say what kind of work it
is for. The one on the desk should take small, safe fixes; the one in the corner should be kept clear of anything risky;
and the queue, which the operator can only reorder, offers no way to express either.

## The outcome

Work that says what kind of work it is makes the queue something the operator can partition instead of merely shuffle.
Tagging pairs with `epic:throttling`: one declares what a runner will take, the other when it will take anything.

## What this epic is not

A project is not a tag. `initiative:projects` stores a project as a record that owns a garden and links sources and
repositories, which a tag cannot.

## Requirements

### Scoped tags on a chunk (product:scoped-tags)

A chunk carries tags such as `type:feature` and `risk:low`. A scope admits one value at a time, so a filter over a scope
can be exhaustive and a chunk cannot quietly be two kinds of thing at once (`decision:tagging/scope-vocabulary`).

Acceptance:

- A chunk never shows two values for the same scope.

### Where a tag comes from (product:tag-sources)

Tags arrive at ingest, are set by the graph, or are placed by the operator. All three are recorded the same way, and the
chunk carries them from ingest to landing.

Acceptance:

- A tag placed at ingest, by the graph, or by the operator is still on the chunk when it lands.

### What a runner will take (product:runner-tag-filter)

Each machine states the tags it will take and the tags it will not, and refusal wins where both could match
(`decision:tagging/untagged-chunks`).

Acceptance:

- A runner never claims a chunk carrying a tag it refuses, even when another of the chunk's tags is one it accepts.

### The partitioned queue, visible (product:partitioned-queue-view)

The board and CLI show the queue as each runner sees it, so an operator can tell starvation from selectivity.

Acceptance:

- The operator sees, on the board and from the CLI, which queued chunks each runner would take.

### Filters on the board and CLI (product:tag-and-item-filters)

The chunk list filters by tag, and by the work item a chunk carries. Someone who thinks in tracker items asks "where is
issue 212?", and the board answers by jumping to its card, Done lane included, instead of leaving them to scan every
lane by eye.

Acceptance:

- Filtering by a tag lists only chunks carrying it, on the board and from the CLI.
- Asking for a work item jumps to its chunk's card, including in the Done lane.
