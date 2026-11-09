---
initiative: adapters
refinement: refined
epics:
  - claude-code
  - opencode
  - opencode-2
  - codex
---

# Adapters

Adapters let the fleet run its workers through more than one coding harness, chosen for each session lineage.

## The problem

An operator should be able to choose the agent that fits the work rather than accept whichever coding harness Blizzard
happened to support first. Today that choice stops at the model: every worker still enters through Claude Code, so a
model available only through another harness is outside the fleet no matter how capable or economical it proves.

## The outcome

The harness becomes a resolved execution capability rather than an identity of the runner. One runner may hold several
harnesses, advertise the ones it can actually operate, and use different ones for different session lineages in the same
chunk. A graph may constrain a lineage to one harness or to an ordered set that accepts several; a chunk may supply the
default where the graph is silent; and the runner supplies the final default. Order makes the choice reproducible: the
first supported acceptable harness wins, never a random draw and never an undeclared substitute.

The initiative lands one harness at a time. The delivered `epic:adapters/claude-code` is the foundation rather than
scope to rebuild. `epic:adapters/opencode`, delivered as the second live binding, was the test of whether the seam is
real, and it built the harness selection every later occupant inherits. `epic:adapters/codex` follows the same contract
after two occupants have forced every Claude-shaped assumption into the binding that owns it. `epic:adapters/opencode-2`
carries the OpenCode binding onto the harness's rewritten 2.x release, proving it afresh rather than assuming the 1.x
contract survived.

## Stance

No harness impersonates another. A capability with no honest equivalent in a harness stays absent and visible as absent;
the platform does not invent a context measurement, reinterpret a compaction threshold, or weaken a permission deny to
claim parity.

No conversation is carried across harnesses. Two harnesses may work the same chunk and the same environments, but they
meet through durable artifacts and the worktree, never by asking one harness to resume another's session.

Automatic cross-harness retry is a later policy, not an accidental consequence of adding a second adapter.
`epic:adapters/opencode` keeps a retry on the harness that began the attempt and escalates when that harness is
unavailable. A future policy may choose another member of an authored acceptable set, but only while minting a fresh
session and only as an explicit, recorded decision.
