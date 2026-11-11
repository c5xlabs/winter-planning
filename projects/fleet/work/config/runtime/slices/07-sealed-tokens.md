---
name: sealed-tokens
external: github:paul-gross/blizzard#2131
depends-on:
  - shared-sealing
  - runner-settings
code:
  - blizzard/src/blizzard/runner/composition.py::held_hub_token
  - blizzard/src/blizzard/runner/hub/held_token.py::HubTokenHolder
  - blizzard/src/blizzard/runner/host_config.py::resolve_token
  - blizzard/src/blizzard/runner/composition.py::DotenvTokenFiles
  - blizzard/src/blizzard/runner/lifecycle/identity_check.py::_take_up_a_corrected_token
  - blizzard/src/blizzard/runner/cli/runners.py::ingest
---

# Sealed tokens

Moves each runner's hub token out of `.env`, where it is keyed by name, into `runner_hub_tokens`, keyed by id and sealed
under the host's key. Adds a way to replace one ([runner-store.md](../spec/runner-store.md) §Sealed tokens, §Replacing a
runner's token, §Seeding step 6). Pre-work for `product:runners-come-and-go`; bound by
`engineering:deprecations-outlive-epic`.

## Change

- `runner_hub_tokens`; the host's key from `BZ_HOST_SECRET_KEY`, or else its minted key directory; the seed's import of
  every file runner's token.
- `HubTokenHolder` opens the row, and re-reads it while identity is in doubt, where today it re-reads `.env`.
- `PUT /api/host/runners/{runner_id}/token` and `blizzard runner token set`, replacing the `.env` correction path.
- Ingest re-seals the moving runner's token under the destination's key.
- `.env`'s runner tokens and `[hub_tokens]` deprecated, read only by the seed, and named in the upgrade note. The note
  and the page on the host's credentials say to back the key up (`engineering:contracts-follow-code`).

## Acceptance

- `blizzard:unit-test`: a token sealed under one runner's id refuses to open under another's.
- `blizzard:component-test` over:
  - an upgraded host whose tokens were in `.env` and `[hub_tokens]` finding every one in the store;
  - a token replaced through the route taken up at the next identity check;
  - a lost key leaving each runner refusing until its token is set;
  - ingest moving a runner whose token re-seals;
  - a rename leaving the runner's token found.
- `blizzard:crash-sweep` with crash points inside the token import, the invariant checker passing at each.
- `blizzard:store-contract`, `blizzard:config-contract`, `blizzard:cli-contract`, `web:client-drift`, and
  `blizzard:compat` pass.
