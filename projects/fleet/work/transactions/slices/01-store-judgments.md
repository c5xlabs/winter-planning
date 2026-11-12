---
name: store-judgments
---

# Store judgments

Hands the four chunk predicates the store judges back to the domain, so the store refuses a chunk write only because it
is stale. Delivers `engineering:store-refuses-stale` for chunks; ordered first by
`decision:transactions/judgments-before-seam`. Spec [store-judgments.md](../spec/store-judgments.md#chunks).

## Change

- `fence` stops reading terminality. Each of the eleven fenced writes takes the domain's terminal verdict and applies it
  after its replay probe, in the order `fence` checks today; `record_lease_minted`'s caller refuses a terminal chunk
  before the write, and `MintAdmission` loses its `terminal` field.
- Each fenced write's domain caller opens `begin()`, declares the chunk, and reads `ChunkFacts` joining the unit;
  `refuse_incoherent_attempt` gains the terminal check.
- The fenced writes that lock a connection they opened, or lock the chunk mid-unit, take
  `transaction: LockedTransaction` instead; `chunk_locked_connection` is deleted. A legacy block reaching a fenced write
  passes its handle's transaction and keeps its opener.
- The backlog write's promoted-chunk filter is deleted; `record_promote` is removed from the port and the store.

## Acceptance

- No chunk write reads a terminal or promoted fact to refuse, to skip, or to choose.
- A replay of a recorded question, artifact, finding delivery, hub step, or escalation on a chunk since made terminal
  returns its recorded outcome; a first write on a terminal chunk is refused with the detail returned today.
- No fenced write locks a chunk on a connection it opened for itself.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
