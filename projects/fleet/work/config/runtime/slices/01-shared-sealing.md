---
name: shared-sealing
---

# Shared sealing

Moves the hub's token sealing into `foundation/` so the runner host can seal its runners' hub tokens with it
([runner-store.md](../spec/runner-store.md) §Sealed tokens). Pre-work for `product:runners-come-and-go`; it changes no
operator surface.

## Change

- The hub's AES-256-GCM sealing, key material, key providers, and rotation move from `hub/secrets/` to
  `foundation/secrets/`.
- The hub's keys, ciphertexts, and behavior are unchanged.

## Acceptance

- `blizzard:unit-test` and `blizzard:component-test` pass over the hub's secrets as they stand, unchanged.
- `blizzard:structural-gate` passes for the new package's layering.
