---
name: host-hub-tokens
depends-on:
  - host-infrastructure
  - epic:config/runtime
code:
  - blizzard/src/blizzard/runner/hub/sealed_tokens.py::HubTokens
  - blizzard/src/blizzard/runner/composition.py::build_hub_tokens
  - blizzard/src/blizzard/runner/composition.py::_running_host_tokens
  - blizzard/src/blizzard/runner/composition.py::build_sealing_hub_tokens
---

# Sealed hub token registrations

Moves the sealed-hub-tokens row in spec [Mixed ownership](../spec/migration.md#area-ownership), delivering
`engineering:area-wiring` and `engineering:one-instance-per-port` under `decision:adopt-di/after-runtime-config`.

## Change

- Register `HubTokens` at host lifetime, replacing `build_hub_tokens` and the daemon's sealing path
  `_running_host_tokens`.
- Fill the `app.state` slot the hub-token routes read from the registered `HubTokens`; the routes migrate in
  `runner-api-host`, where they iterate `ServedRunners`.
- `build_sealing_hub_tokens` is called from `runner/cli/runtime.py`: by `token set` on a stopped host, and by
  `_seed_entries`, the per-runner seed that `StoreHostedRunners.built` runs on every reconcile of the running host. The
  seed resolves the registered `HubTokens` once `hosted-runners` registers the seeder; until `token set` migrates,
  `build_sealing_hub_tokens` stays a bounded compatibility entrypoint over the registered provider
  ([Mixed ownership](../spec/migration.md#area-ownership)). `HostKeyCiphers` and `DotenvTokenFiles` serve only
  `build_runner_ingest` and move with the ingest graph in `cli-runner-writes`.

## Acceptance

- `blizzard:component-test`: a token set on a running host reaches its runner within its next check cycle; a token
  sealed under the host key reads back; a missing or wrong host key refuses as it does today.
- `blizzard:unit-test` (`tests/unit/runner/cli`): `token set` on a stopped host behaves unchanged through the
  compatibility entrypoint.
- `blizzard:component-test`: one `HubTokens` serves the host, every runner and, once `hosted-runners` lands, the
  per-runner seed; `blizzard:typecheck` and `blizzard:unit-test` (the host graph/identity guards in `tests/guards`)
  pass.
