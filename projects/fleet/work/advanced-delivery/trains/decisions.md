# Trains decisions

## Split by arrival order (decision:advanced-delivery/trains/split-by-arrival-order)

**Decider:** engineering

A failing batch is split by arrival order and only a prefix of it ever lands, so the base takes the members in the order
they arrived, and a failure two members cause together is charged to the later of them, against a base that already
holds the earlier.

## Remembered refused advance (decision:advanced-delivery/trains/remembered-refused-advance)

**Decider:** engineering

An advance the forge refuses as protected is remembered on the repository's probe and leaves `refs.advance` out until an
operator takes a probe on demand, however many scheduled probes follow, because classic branch protection is unreadable
in detail without admin access and a train refused once must never loop into the same refusal.

## Forge-queue fallback (decision:advanced-delivery/trains/forge-queue-fallback)

**Decider:** engineering

`forge-queue` is the one policy that declares a fallback, `merge-on-checks` unbatched, and setting it on a repository
whose probe shows no queue is accepted with a warning rather than refused. The fallback is applied before any capability
is checked, so the check before a pass never refuses a repository for the queue it is about to do without, and a
repository whose queue is switched off lands each change alone.
