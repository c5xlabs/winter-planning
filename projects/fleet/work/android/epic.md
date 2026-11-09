---
epic: android
refinement: outline
status: planned
---

# Android app

A standalone epic; its requirements are inline below.

## The problem

An operator who follows more than one fleet has no single place to see them. `epic:pwa` puts one hub on a home screen,
which is most of what a phone needs, and stops exactly where a person running a work fleet and a personal one begins to
want them side by side.

## The outcome

A native application is the shape that can hold that: every hub the operator follows registered the way a Jira app
registers its sites, local runners alongside them, and all of it read as one.

This enters as an exploration, a deliberate maybe. What a native app must earn is the multi-hub view and the
notification reliability a wrapped page cannot give.

## Requirements

### Hub and runner registry (product:hub-registry)

The operator adds each hub they follow and each runner they own, authenticating to each separately, and the app
remembers them. The registry is designed once and shared with `epic:ios`; whichever ships first decides it.

Acceptance:

- The operator registers a hub or a runner once, authenticating to it, and finds it still registered when they next open
  the app.

### One view across fleets (product:unified-view)

Chunks, questions, and escalations across every registered fleet read as a single list rather than a set of tabs, which
is the whole argument for building this at all. The aggregation model is shared with `epic:ios` the same way the
registry is (`decision:android/aggregation-site`).

Acceptance:

- Chunks, questions, and escalations from every registered hub appear in one list, not one tab per hub.

### Native notifications (product:native-notifications)

The fleet's voice arrives the way the phone's other notifications do, reliably enough to be trusted while the operator
is asleep.

Acceptance:

- The fleet's notifications appear as the phone's own notifications do.

### Acting from the app (product:act-from-app)

The operator answers a question and unblocks a chunk from the app, since a notification that cannot be acted on is only
an interruption.

Acceptance:

- The operator answers a question from the app.
- The operator unblocks a chunk from the app.

## Open questions

- **Product owner:** What parity with the board costs to maintain, and which board surfaces the app deliberately refuses
  to carry.
- **Product owner:** Whether the multi-hub view is genuinely wanted by enough operators to justify a native application,
  which is the exploration's actual question.
