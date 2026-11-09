---
epic: documentation
refinement: outline
status: planned
---

# Documentation site

A standalone epic; its requirements are inline below.

## The problem

Blizzard is public and installable, and its README is the entire published surface, so every question past the first one
costs a clone. An operator standing a fleet up, a graph author deciding what a node may declare, and an engineer
weighing whether blizzard fits their shop all arrive with different questions, and today each of them answers theirs by
reading the repository.

## The outcome

They have somewhere to be sent instead: a documentation site at `paul-gross.github.io/blizzard-docs/`, structured the
way [winter's](https://paul-gross.github.io/winter-docs/) is. Winter's site is the working pattern to copy rather than
re-derive.

## Requirements

### A site of its own (product:docs-site)

The documentation is published at `paul-gross.github.io/blizzard-docs/` and releases on its own cadence rather than
blizzard's.

Acceptance:

- The site is reachable at its address.
- A documentation change publishes without waiting for a blizzard release.

### Getting started (product:getting-started)

One route leads from nothing to a fleet that has worked a chunk, written for the reader who has not decided yet whether
to commit an evening to this.

Acceptance:

- A reader following the route alone ends with a fleet that has worked a chunk.

### Operating a fleet (product:operator-guide)

The site covers standing up a hub, running a runner, and what the operator is responsible for once both are alive.

### CLI and configuration reference (product:cli-reference)

A browsable reference covers the CLI and configuration, generated from the source of truth wherever it can be, because a
reference written by hand is a reference that will disagree with the software.

### Concepts for graph authors (product:graph-concepts)

The site explains what a node may declare and what the platform does with it, the questions a graph author currently
answers by reading the schema.

### The site as one file an agent can read (product:agent-readable-site)

An agent reads the whole site as one file, as easily as a person reads one page.

## Open questions

- **Engineering:** How much of the reference is generated versus written, and what check keeps the written part honest
  as the software moves.
- **Product owner:** What stays in the repository README once the site exists, which should be the shortest possible
  road to the site.
