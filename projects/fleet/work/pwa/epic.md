---
epic: pwa
refinement: outline
status: planned
---

# Board on the home screen

A standalone epic; its requirements are inline below.

## The problem

The fleet's most time-sensitive moments happen when `persona:product-owner` is nowhere near a browser. A chunk that
stopped at nine in the evening waits until someone next walks past a terminal, and the board, the one surface that would
tell them, lives on a laptop that is closed.

## The outcome

The cheapest honest answer is to put the board on the home screen: a progressive-web-app shell, laid out for a thumb,
with questions arriving as notifications.

This is one of two answers to the same need, weighed against `epic:android` and `epic:ios`. Which one blizzard invests
in is a deliberately deferred choice (`decision:mobile-path`).

## Depends on

- `epic:chat`, whose notification fan-out this epic adds a channel to rather than a mechanism.

## Requirements

### An installable board (product:installable-board)

The board installs to a home screen and launches like an application, without a second frontend to maintain.

Acceptance:

- The operator adds the board to a phone's home screen and opens it like an application.

### A layout for a thumb (product:thumb-layout)

The phone shows the few things that matter there: how the night is going, what is waiting on a person, and one chunk in
detail (`decision:pwa/phone-layout`).

Acceptance:

- On a phone, the operator sees how the night is going, what is waiting on a person, and one chunk in detail.

### Question notifications (product:question-notifications)

Questions arrive as notifications through the web, through the same fan-out `epic:chat` builds.

Acceptance:

- A question the fleet raises arrives on the phone as a notification.

### Honesty when stale (product:stale-warning)

A phone is offline often; a board that is showing an old picture says so rather than letting an operator act on it.

Acceptance:

- A board showing data that is no longer current says so on screen.

## Open questions

- **Engineering:** What web push can actually promise on iOS, which is the constraint that decides whether this is
  sufficient or merely cheap.
- **Product owner:** Whether answering a question from the shell is in scope here or stays with the channels that
  already do it well.
