---
name: store-judgments
---

# Store judgments

Hands the four domain predicates the store judges back to the domain, so the store refuses a write only because it is
stale. Delivers `engineering:store-refuses-stale`; ordered first by `decision:transactions/judgments-before-seam`. Spec
[store-judgments.md](../spec/store-judgments.md).

## Change

- The terminal check inside `fence`, the escalation that runs that fence on its own write, the backlog write that drops
  a promoted chunk's position, and the promote that writes nothing for an already-promoted chunk each return their fact
  to the domain, which decides.
- The store conditions each write on the token the domain read, taking the compare-and-set form where one version or
  monotonic token covers the decision.
- Leaves the locked seam and its callers alone.

## Acceptance

- No store write reads a status-family fact to refuse.
- A write conditioned on a stale token is refused; one on the current token succeeds.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
