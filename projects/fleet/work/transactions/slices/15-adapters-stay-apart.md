---
name: adapters-stay-apart
depends-on:
  - store-judgments
---

# Adapters stay apart

Hands every write an adapter makes to another repository's facts, through a sibling's private helper, back to the
domain. Delivers `engineering:adapters-stay-apart`. Spec [adapters.md](../spec/adapters.md).

## Change

- The domain composes both repositories in one unit opened through `IHubTransactions.begin`.
- `WorkItemStore`'s composite mints are moved to the domain.

## Acceptance

- No adapter calls a sibling adapter's private helper to write that sibling's facts.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
