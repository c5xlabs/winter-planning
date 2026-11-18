---
name: env-bands
depends-on:
  - layered-load
code:
  - winter-workspace/crates/core/src/config.rs
  - winter-workspace/crates/core/src/termination.rs::ConfigError
---

# The value rules of the env bands

Adds to the load the `env.*.vars` rows of Tables, the env command-entry table, the `[env.vars]` row of Removed keys, and
the `ManagedName` cross-key rule of [configuration-model.md](../spec/configuration-model.md), at the env-band place of
step 8 of the Load order. Delivers the closed-set bullet and the removed-key bullet of
`product:configuration-validation` for the env bands. Band resolution, the token grammar, and running a command entry
stay with `epic:running-environments/scope-environment`.

## Change

- `ConfigError` gains `EnvVar` with `EnvVarFault` as the spec's Domain gives it, naming the band and the key.
- Under `env.workspace.vars`, `env.feature.vars`, and every other `env.<name>.vars`, a value that is a string, an
  integer, a float, or a command-entry table is accepted; a boolean, an array, or another type is
  `EnvVarFault::UnsupportedType`.
- A command-entry table is checked against its closed field set: an unknown field, a missing or empty `command`, a
  `format` other than `raw`, `dotenv`, or `json`, a non-boolean `shell`, and an `exports` that is not an array of
  strings are each their `EnvVarFault`. A command entry under a key that starts with `WINTER_` is `ManagedName`.
- `[env.vars]` in the merged document is `RemovedKey::EnvVars`, listing the keys found and naming both replacement
  tables, met with the env bands.
- No value is kept and no command entry runs.

## Acceptance

- Unit tests, one per `EnvVarFault` variant, each naming the band and key: `X = true`; `X = [1]`;
  `X = { command = "c", extra = 1 }`; `X = { format = "raw" }`; `X = { command = "c", format = "yaml" }`;
  `X = { command = "c", shell = "yes" }`; `X = { command = "c", exports = "A" }`; `WINTER_X = { command = "c" }`.
- Unit tests: `X = "a"`, `X = 3`, `X = 1.5`, and `X = { command = "c" }` load under `workspace`, `feature`, and a band
  named `alpha`; `WINTER_X = "a"` loads.
- A unit test: `[env.vars] A = "1"` and `B = "2"` is `RemovedKey::EnvVars` listing `A` and `B`.
- A unit test with an invalid env band and an invalid model tier reports the env band, the earlier part of step 8.
- A test that a command entry's `command` is never run: a load over `X = { command = "touch <marker>" }` leaves no
  marker file.
- `cargo dev gate full` exits 0.

## Out of scope

- Model tiers and agent overrides, later in step 8: `model-tiers`.
- Resolving a band to variables: `epic:running-environments/scope-environment`.
