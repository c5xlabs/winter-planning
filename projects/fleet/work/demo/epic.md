---
epic: demo
refinement: outline
status: planned
---

# Demo mode

A standalone epic; its requirements are inline below.

## The problem

Everything blizzard does begins with a credential. A newcomer who wants to see what a fleet actually does must first
obtain a coding-harness subscription, a forge token, and a repository worth pointing at, which is a great deal of trust
to ask of someone who has not yet seen the product work. A graph author has a quieter version of the same problem: the
only way to find out what their graph does is to spend a night and some money finding out.

## The outcome

Blizzard is runnable with nothing real attached. The mock fleet already exists for blizzard's own tests, with a mock for
every seam, and today's mocks execute scripted prompts. What it lacks is a harness that answers like one, a forge with a
backlog, and a way to run it as blizzard itself.

## Requirements

### A harness that answers plausibly (product:plausible-harness)

The demo's harness responds to a real prose prompt in a way that looks like work, deterministically and at no cost.

Acceptance:

- The same prompt gets the same response every time.
- Running the demo costs nothing.

### A mock forge holding a backlog (product:mock-backlog)

The demo's forge holds items to browse, ingest, and land against, so the whole path from intake to delivery is walkable.

Acceptance:

- A newcomer takes an item from the mock backlog through ingest to landing with no real credentials.

### Demo as a mode of blizzard (product:demo-mode)

The real hub, runner, graphs, and board run above mock seams, packaged as a way to run blizzard rather than as a sibling
checkout that will drift (`decision:demo/mode-selection`).

Acceptance:

- The demo runs from an ordinary blizzard install with no harness subscription, forge token, or repository.

### Named scenarios (product:demo-scenarios)

Named scenarios play out over time: a good night, a chunk that wedges, a question that needs answering. A newcomer sees
the fleet's interesting moments in minutes instead of waiting for one to happen.

Acceptance:

- A newcomer picks a scenario by name and sees its moment within minutes.

## Open questions

- **Product owner:** How plausible the mock harness must be to be worth having, and whether plausibility requires a
  model at all or is better served by something wholly deterministic.
- **Product owner:** What keeps the scenarios honest as the graph evolves, since a demo that shows last quarter's fleet
  is worse than no demo.
