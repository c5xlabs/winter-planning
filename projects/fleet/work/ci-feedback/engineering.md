# CI feedback engineering

Engineering for `epic:ci-feedback`.

## What already holds

- The delivery machinery this feeds into exists: the merge queue, per-repo landed facts, and gated pull requests.

## Brief requirements

- engineering:delivery-session-mapping — One stored row links a chunk's delivery (its PR, where a gate opened one) to
  the session that receives its feedback; it is never inferred from history. Serves `product:feedback-routed-home`.

## Open questions

- **Engineering:** How much of a failing run the session is fed: the excerpt heuristic that is informative without
  drowning the context.
