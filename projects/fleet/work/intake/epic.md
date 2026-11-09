---
epic: intake
refinement: outline
status: planned
---

# Backlog intake

A standalone epic; its requirements are inline below.

## The problem

Queuing work today means having the id in hand. The operator reads the backlog in the forge, copies an identifier, hands
it to blizzard, and repeats, which is fine for one chunk and absurd for an afternoon of triage.

## The outcome

Working a backlog down is a different act from queuing a chunk, and it gets a surface built for it: somewhere the
operator browses what is open, chooses several, and shapes what lands.

## What this epic is not

It never defines work. The forge stays the backlog's home, and this screen only moves its items into the fleet with less
friction.

## Requirements

### An intake workbench (product:intake-workbench)

The board's per-card grouping controls grow into a screen of their own: queued and candidate work side by side,
inspected, grouped, ungrouped, and sent through ingest in one sitting.

Acceptance:

- The operator sees queued and candidate work side by side on one screen.

### Browsing the backlog (product:backlog-browsing)

The operator browses and searches the open items of a work source instead of arriving with an id
(`decision:intake/source-enumeration`, `decision:intake/search-scope`). A source that cannot be browsed still ingests by
id, and the screen says so honestly.

Acceptance:

- The operator finds an item to ingest without knowing its id.
- For a source that cannot be browsed, the screen offers ingest by id and says why browsing is unavailable.

### Many at once (product:batch-ingest)

The operator selects several items and pulls them through ingest together, then groups what belongs together before any
of it is claimed.

Acceptance:

- Several selected items go through ingest in one action.
- Items ingested together can be grouped before any runner claims them.

### Ungrouping (product:ungroup-chunk)

The operator splits a folded chunk back apart, which grouping cannot do today and which an operator who grouped hastily
needs immediately (`decision:intake/ungroup-shape`).

Acceptance:

- A folded chunk is split back into its parts from the workbench.

## Open questions

- **Product owner:** How much of the forge's own item detail this surface shows before it starts re-implementing the
  forge.
