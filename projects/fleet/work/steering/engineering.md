# Steering a running chunk engineering

Engineering for `epic:steering`.

## What already holds

- The runner's at-most-once resume already interrupts a dormant session with a system-authored message.

## Brief requirements

- engineering:human-resume-message — A steering message rides the runner's at-most-once resume; this epic adds a human
  author and the path that carries their words to it. Serves `product:pause-delivery`.

## Open questions

- **Engineering:** What a natural pause is in each harness, since the answer is the whole delivery guarantee and no two
  harnesses expose it the same way.
