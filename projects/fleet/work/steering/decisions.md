# Steering a running chunk decisions

## A message for a finished chunk (decision:steering/finished-chunk-message)

**Open.** What becomes of a message for a chunk that finishes before it lands: dropped with a note, or surfaced as
something the operator must decide about; it blocks `product:pause-delivery`.

## A second message (decision:steering/second-message)

**Open.** Whether a second message queues behind the first or replaces it, given that an operator who is steering is
usually correcting themselves; it blocks `product:pause-delivery`.
