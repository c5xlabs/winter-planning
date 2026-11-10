# Experiment decisions

## Trial work items (decision:lab/experiment/trial-work-items)

**Open.** Whether a trial, which deliberately wraps no work item where every chunk today wraps one or more, is a chunk
with an empty work list or wraps a hub-authored item the experiment owns that never closes, which decides how much of
the chunk's own rules bend for the lab; it blocks `product:trial-chunks`.

## One queue for trials (decision:lab/experiment/one-queue)

Trials travel the ready queue everything else does, and only a runner that brings a lab workspace claims one. A separate
queue would need claiming rules of its own; a runner that declares what kind of workspace it offers needs only the claim
it already makes.
