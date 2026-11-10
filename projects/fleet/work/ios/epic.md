---
epic: ios
refinement: outline
status: planned
---

# iOS app

A standalone epic; its requirements are inline below.

## The problem

An operator who follows more than one fleet on Apple's platform has the same problem `epic:android` answers on Android:
no single place to see them all.

## The outcome

Every hub and runner the operator follows is registered in one place, read as a single view of chunks, questions, and
escalations, and reached through notifications that behave like the platform's own. It enters as an exploration
alongside its Android counterpart, and it shares that epic's registry and aggregation design; whichever ships first
decides them. What earns it a separate epic is that the platforms differ in exactly the places this work lives: the
store, the notification plumbing, and what an application is permitted to do in the background.

## Requirements

### The shared model on iOS (product:shared-model-on-ios)

The hub-and-runner registry and the unified cross-fleet view of `product:android/hub-registry` and
`product:android/unified-view` work on Apple's platform, built against it rather than redesigned for it.

Acceptance:

- The operator registers hubs and runners and reads them as one list, as on Android.

### Notifications the operator can trust (product:ios-notifications)

Notifications arrive through Apple's push service, reliably enough that an operator can stop watching and trust the
phone to tell them.

Acceptance:

- The fleet's notifications arrive as the phone's own notifications do.

### Acting from the notification (product:act-from-notification)

The operator answers a question and unblocks a chunk without a trip to a laptop.

Acceptance:

- The operator answers a question from the phone.
- The operator unblocks a chunk from the phone.

### Background behaviour that survives review (product:review-safe-background)

What the app may do while closed is decided against Apple's rules rather than discovered by rejection.

Acceptance:

- What the app does while closed is written down against Apple's rules before it is submitted.
- The app is not rejected at review for its background behaviour.

## Open questions

- **Product owner:** What App Store review makes of an application whose entire purpose is operating a fleet of coding
  agents, and what the listing has to say to pass.
- **Engineering:** How much implementation is genuinely shared with `epic:android` beyond the design, and whether a
  shared layer pays for itself at two platforms.
- **Product owner:** Whether iOS's background rules permit the freshness the unified view implies, or whether the app is
  honest about being a notification client with a good list.
