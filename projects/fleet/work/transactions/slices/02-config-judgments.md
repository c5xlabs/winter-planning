---
name: config-judgments
---

# Config judgments

Hands the forge and secret rules the store judges back to the domain, so the store refuses a config write only because
it is stale. Delivers `engineering:store-refuses-stale` for forges and secrets. Spec
[store-judgments.md](../spec/store-judgments.md#forges-and-secrets).

## Change

- A repository or work-source write, and a forge or secret retirement, open one unit declaring the forge and secret
  keys they decide on, and the domain judges retirement and referrers under those locks.
- The store's no-op row locks on forges and secrets give way to the declared keys.
- `referred_forge`, `_check_secret`, and the referrer checks inside `record_forge_lifecycle` and the secret's
  `record_lifecycle_fact` return their facts and no longer refuse.

## Acceptance

- No forge or secret write reads a retirement or referrer fact to refuse.
- A repository or work source naming a retired forge or secret is refused, and a forge or secret in use is not retired,
  with the refusals returned today.
- Behavior is unchanged at every route and command (`engineering:behavior-preserving-landings`).
