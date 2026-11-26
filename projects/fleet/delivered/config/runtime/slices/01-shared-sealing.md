---
name: shared-sealing
external: github:paul-gross/blizzard#2125
code:
  - blizzard/src/blizzard/hub/secrets/internal/aes_gcm.py::AesGcmSecretCipher
  - blizzard/src/blizzard/hub/secrets/internal/key_material.py::generation_of
  - blizzard/src/blizzard/hub/secrets/internal/env_keys.py::EnvKeyProvider
  - blizzard/src/blizzard/hub/secrets/internal/directory_keys.py::DirectoryKeyProvider
  - blizzard/src/blizzard/hub/secrets/rotation.py::rotate_keys
---

# Shared sealing

Moves the hub's token sealing into `foundation/secrets/` so the host can seal its runners' hub tokens
([runner-store.md](../spec/runner-store.md) §Sealed tokens). Pre-work for `product:runners-come-and-go`; it changes no
operator surface.

## Change

- The hub's AES-256-GCM sealing, key material, key providers (`env_keys.py`, `directory_keys.py`), and rotation move
  from `hub/secrets/` to `foundation/secrets/`.
- The hub's keys, ciphertexts, and behavior are unchanged.

## Acceptance

- `blizzard:unit-test` and `blizzard:component-test` pass over the hub's secrets, the tests changing only their imports.
- `blizzard:structural-gate` passes for the new package's layering.
