---
epic: ci-feedback
refinement: outline
status: planned
---

# CI feedback

A standalone epic; its requirements are inline below.

## The problem

When CI goes red or a reviewer asks for changes after a delivery is open, the news has to reach the agent that did the
work, and today the messenger is a human. The failure escalates with a resume command, and someone pastes it, relays the
log, and shepherds the fix. That human relay is the version that ships; it is correct, and it does not scale past the
operator's patience.

## The outcome

This epic automates the relay: the result finds its own way back into the owning session. The human relay never stops
working, and it takes over whenever the automated one gives up.

## Requirements

### Feedback routed home (product:feedback-routed-home)

A red CI run or a change request lands back in the session that owns the delivery. The dormant session is resumed with
the failure in hand, a log excerpt or the review comments, the same way an answered question resumes its asker.

Acceptance:

- After CI goes red on an open delivery, the owning session resumes with an excerpt of the failing log, with no one
  pasting anything.
- After a reviewer requests changes, the owning session resumes with the review comments.

### Capped feedback rounds (product:feedback-round-cap)

Feedback rounds are capped at two. An agent that fixes one thing and breaks another can oscillate indefinitely; the cap
turns oscillation into an ordinary escalation, and the human relay takes it from there.

Acceptance:

- Feedback arriving after the second round escalates to the operator instead of resuming the session.
