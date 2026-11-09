# Chokepoint requirements

The requirements of `epic:chokepoint/chokepoint`.

## Mark a chunk as a chokepoint (product:chokepoint-marking)

Being a chokepoint is a property of the chunk, which the operator sets and clears the way they set its urgency.

Acceptance:

- The operator can mark a chunk as a chokepoint, and clear the mark, from the board.
- The operator can do the same from the CLI.

## A chokepoint you can see coming (product:chokepoint-indicator)

An operator scanning the queue spots the chokepoint ahead the way a driver spots a lane closure sign: early enough to do
something about it.

Acceptance:

- A chokepoint chunk carries an indicator of its own on the board — an icon, a color, a flag — that no ordinary chunk
  carries.
- The indicator appears on the chunk's card wherever the card appears.
- The indicator appears from the moment the chunk is marked, not only once it starts.

## Move a chokepoint (product:chokepoint-moving)

Deciding to do the big thing sooner or later is a drag on the board, not a negotiation with the fleet.

Acceptance:

- The operator can move a chokepoint up or down the queue from the board.
- For each move, the board shows what it means: which chunks the chokepoint would now wait on, and which it would hold
  back.

## Rules kept at the moment of claiming (product:claim-time-enforcement)

A chokepoint is enforced when a runner goes to pick up work, against the queue as it stands at that moment, and nowhere
else. Until it starts it never freezes the queue in front of it, and once it starts the road is closed until it clears,
because the chokepoint exists so that nothing new starts beside it.

Acceptance:

- Nothing behind a chokepoint is claimed until the chokepoint is done.
- The chokepoint is not claimed while any chunk ahead of it is still open.
- Before the chokepoint starts, anyone can put work ahead of it — minting a chunk ranked ahead of it, moving one past
  it, or giving one behind it more urgency than the chokepoint carries — and the chokepoint waits for that work too.
- Once the chokepoint is claimed, nothing else in its project is claimed until it is done, however urgent and wherever
  it is ranked; an urgent fix that arrives while the big change is running waits at the head of the line.

## Open work ahead holds the chokepoint (product:open-work-ahead-holds)

Anything open ahead of a chokepoint holds it back, whatever state it is in, and the fleet never routes around it.

Acceptance:

- A chunk ahead that is running, paused, parked at a gate, or waiting on an answer still holds the chokepoint back.
- The board names each chunk standing in the chokepoint's way.

## Ahead and behind follow claim order (product:claim-order-placement)

Under `epic:urgency`, ahead and behind follow claim order — the most urgent group first, rank within each — the same
order a runner reaches for work in, so a chokepoint's own urgency decides how high the wall stands. An operator planning
a rewrite can mark the chokepoint `standard` and still let an urgent fix through on the night it arrives, without anyone
moving the fix by hand, or mark it with their most urgent group to hold back everything.

Acceptance:

- Everything before the chokepoint in claim order finishes before it is claimed, and everything after it waits for it to
  clear.
- A steady stream of work more urgent than the chokepoint keeps it waiting until the operator changes its urgency.
- A chunk less urgent than the chokepoint waits behind it even when it is ranked ahead, so a background job at the top
  of the queue does not hold up a `standard` rewrite; raising that chunk's urgency to match puts it ahead.

## Several chokepoints pass one at a time (product:chokepoints-in-turn)

A project may hold more than one chokepoint, and ordinary work between them flows as usual.

Acceptance:

- Chokepoints pass one at a time, in claim order, so an urgent chokepoint ranked behind a `standard` one goes first.
- No two chokepoints can each stand ahead of the other, because claim order is a single line.

## A chokepoint narrows only its project (product:project-scoped-chokepoint)

Once `initiative:projects` gives the hub more than one project, a chokepoint narrows only the project its chunk belongs
to, because a rewrite of one codebase has no reason to stall a project that shares none of it. Until projects land, the
hub holds a single project, and a chokepoint narrows the whole fleet.

Acceptance:

- Ahead and behind are counted within the chokepoint's project's queue.
- Runners go on claiming every other project's work as if nothing had happened.

## A failed chokepoint keeps the road closed (product:failure-keeps-road-closed)

Releasing the fleet onto a half-finished sweep is exactly the merge a chokepoint exists to prevent, so a chokepoint that
escalates keeps the road closed until a human intervenes.

Acceptance:

- While an escalated chokepoint awaits a human, nothing ranked behind it is claimed.
- Resolving and resuming the chokepoint, or stopping it, reopens the road; stopping it ends the chokepoint and lets the
  work behind it go.

## The narrowing is visible (product:visible-narrowing)

An operator should never mistake a deliberate drain for a wedged fleet.

Acceptance:

- The board shows that the fleet is narrowing toward a chokepoint.
- The board shows which chunks the chokepoint is still waiting on.
- The board shows why runners are taking nothing from that project.
