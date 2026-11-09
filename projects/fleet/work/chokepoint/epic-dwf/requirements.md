# Epic-level development workflow requirements

The requirements of `epic:chokepoint/epic-dwf`.

## Epic-level change landed as one (product:epic-level-change)

A change too large for one session and too interlocked to split into separate chunks is planned whole, built by a swarm,
and lands as a single change.

Acceptance:

- The graph plans the change as a whole and divides it into pieces that can be built side by side.
- A swarm of agents works the pieces at once.
- Their work comes back together into a single change that is verified, reviewed, and landed as one.

## Small adjustments fanned out (product:small-adjustment-fan-out)

Thousands of small edits, each best made by an agent looking only at its own corner, are fanned out across the swarm,
and the lane proves afterwards that the sum of many small edits still behaves like the system it replaced.

Acceptance:

- Each edit is made by an agent working only its own corner of the codebase.
- Before the change lands, the lane shows the changed system still behaves like the system it replaced.

## The swarm runs inside one node (product:swarm-in-one-node)

The traditional lanes go back and forth by design: a plan bounces off its gate, review sends the build round again, and
every station is a separate session so that its judgment stays independent of the work it judges. An `epic-dwf` chunk
does not travel that way: one session takes the chunk, reads the documents that govern the work, and runs the swarm from
there as a dynamic workflow, which is how the architectural sweep was delivered.

Acceptance:

- The session plans, fans out, verifies, reviews, and assembles the change without handing the chunk back to the graph
  until it is ready to land.
- The review is designed by the session that ran the build, at a scale no single review node could reach, and runs
  inside the swarm alongside everything else.
- The graph around the swarm holds only the node, the questions it raises, and the landing.

## Practices molded from the sweep (product:sweep-practices)

The account the architectural sweep's orchestrating session wrote afterwards,
[Nine Sweeps, One Line](../presentations/nine-sweeps-one-line.md), is this epic's reference, and the node's prompt is
molded from it, practice by practice.

Acceptance:

- Agents are disposable and hand off before their context runs long.
- Scripted moves are kept apart from judgment, so they can be replayed onto a base that moved.
- Every new rule is paired with a gate that enforces it and a cold evaluation that proves agents can find it.
- Each reviewer is paired with a skeptic told to refute it.
- Every decision the human made gets an itemized audit, so that nothing quietly becomes a follow-up.

## Decisions gathered before code moves (product:decisions-up-front)

The operator answers the swarm's decisions together rather than meeting them one at a time through the night; the sweep
turned a hundred and forty-eight hidden decisions into ninety-six questions answered in a single reply.

Acceptance:

- Before any code moves, the swarm surveys the change and gathers every decision the governing documents cannot settle
  into one list for the operator.

## A person approves the landing (product:landing-approval)

Work this size goes out under a person's name, so the operator's second deliberate moment comes at the end.

Acceptance:

- Before the change lands, the swarm stops and asks the operator.

## Chosen, never triaged (product:chosen-lane)

An operator who sends work down this lane is also deciding to stop the fleet for it, and that is a decision a person
makes deliberately, not one a classifier makes on their behalf.

Acceptance:

- Triage never routes a chunk to `epic-dwf`.
- Choosing `epic-dwf` for a chunk and marking it a chokepoint are the same act.

## Each run remolds the prompt (product:retrospective-remolding)

The lane closes with a retrospective, the way `adv-dwf` does, and here it carries more weight than anywhere else: the
reference records one delivery, and some of what worked there belonged to that epic rather than to the lane.

Acceptance:

- Each run's retrospective writes its own account of what the swarm did, what held, and what it would change.
- The node's prompt is molded again from what those accounts have in common.

## A swarm-capable harness (product:swarm-capable-harness)

The node runs on a harness that can orchestrate a swarm of its own, and today that means Claude Code, whose dynamic
workflows drove the architectural sweep. The requirement is the capability, not the vendor, and `milestone:polyglot`
already lets a node choose its harness.

Acceptance:

- The node declares that it needs a swarm-capable harness, and runs only on one.

## A spend ceiling inside the node (product:in-node-spend-ceiling)

**Proposed.** A swarm spends its whole budget inside one node, over the better part of a day — the architectural sweep
ran some four hundred and fifty agents — while today's spend controls were built for chunks that move from node to node:
a chunk's cap is checked at the boundary between one node and the next, and a runner's rolling ceiling stops the runner
claiming more work without stopping the session already running, so neither would interrupt a swarm once it started. The
agent observer of `initiative:code-harness-window` is what would let a ceiling see the spend at all; what the swarm may
spend is `decision:chokepoint/epic-dwf/swarm-spend`.

Acceptance:

- The operator sets the ceiling before the swarm starts.
- The ceiling is honored while the swarm runs.

## Open questions

- Whether Codex, OpenCode, or any other harness blizzard runs can now orchestrate parallel agents inside one session,
  and well enough to carry a change this size; research answers it as the first piece of work when the epic is picked
  up.
