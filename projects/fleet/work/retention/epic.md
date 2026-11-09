---
epic: retention
refinement: outline
status: planned
---

# Retention

A standalone epic; its requirements are inline below.

## The problem

The hub keeps everything. Every fact, transcript, artifact, question, and measurement it has ever received is still
there, which is a comfortable arrangement for about a year and then arrives one morning as a daemon that will not start.
The operator never chose this; storage was simply assumed to be free. This is far cheaper to design now than after there
are years of facts to move.

## The outcome

The operator holds the decision: each kind of exhaust is kept for as long as they find it useful, and then let go.

## Requirements

### A policy per class of exhaust (product:retention-policy)

Facts, transcripts, artifacts, questions, and measurements age out on their own terms, because they are useful for very
different lengths of time. The operator states each one; the platform stops guessing (`decision:retention/expiry-form`).

Acceptance:

- The operator sets how long each class is kept, and records older than that are let go.

### Nothing live is forgotten (product:derivation-safe-expiry)

What leaves is only ever what nothing derives from. A record some live status is computed from is never let go,
regardless of age, so the fleet cannot forget its way into lying.

Acceptance:

- A live status still shows correctly after every record older than its class's policy has been let go.

### Export before drop (product:export-before-drop)

`initiative:fact-egress` decides what has already left the building; nothing is dropped that the operator asked to keep
and has not yet been handed.

Acceptance:

- A record the operator asked to export is not dropped before it has been exported.

## Open questions

- **Product owner:** The granularity a policy is worth stating at, per class alone or per class and project, and whether
  an operator thinks in ages or in sizes.
- **Product owner:** What a partially-retained chunk shows when someone opens it a year later, since a detail page whose
  transcript is gone needs an answer better than a blank panel.
