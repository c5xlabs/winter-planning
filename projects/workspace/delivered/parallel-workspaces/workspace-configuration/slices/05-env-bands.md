---
name: env-bands
depends-on:
  - load-seam
---

# The `[env]` bands are read and refused at load

Fills `config_model/env_bands.rs` with the `env.*.vars` rows, the command-entry table, the number rendering, and the
removed `[env.vars]` of [configuration-model.md](../spec/configuration-model.md). Serves
`product:configuration-validation` and `engineering:inert-configuration-read`.

## Change

- The `workspace`, `feature`, and named bands are read into `EnvBands`; a string is a literal, an integer and a float
  become the string the model's rendering block gives, a table is a command entry with its closed field set, and every
  other value is `EnvVar` with its fault. `[env.vars]` is `RemovedKey::EnvVars` carrying the keys found.
- An `env` that is not a table, a band that is not a table, and a `vars` that is not a table each read as absent, with
  no message, as the spec's block under Tables states from the reference source.
- No command of an entry is run and no `EnvVar` failure carries the value of an entry.
- Recorded scenarios, each plain, over fixture workspaces in temporary directories: `ws-index/env-vars-removed`,
  `ws-index/env-boolean-value`, `ws-index/env-command-unknown-field`, `ws-index/env-command-under-managed-name`,
  `ws-index/env-command-entries-not-run`, whose command would create a file in the workspace, and
  `ws-index/env-is-string`, whose shared file alone holds `env = "x"`.
- Files touched: `crates/core/src/config_model/env_bands.rs`; `scenarios/ws-index/**` for the six scenarios.

## Acceptance

- Unit tests assert each `EnvVarFault`, and that a named band is any sub-table other than `workspace` and `feature`.
- A unit test asserts empty bands and no error for `env = "x"`, for `env.feature = "x"`, and for
  `env.feature.vars = "x"`.
- A unit test table of literal pairs asserts the rendering of `1.0`, `-0.0`, `0.1`, `1e16`, `1e15`, `0.0001`, `0.00001`,
  `1.5e300`, `inf`, `-inf`, `nan`, and the largest and smallest 64-bit integers.
- `cargo scenarios ws-index` passes, the six scenarios against the record of the reference tool with no deviation; the
  tree of `env-command-entries-not-run` is unchanged by the run. The record of `env-is-string` shows exit code 0; if it
  shows exit code 1, the change stops and reports it.
- `cargo dev gate slice` passes; no file outside those named changes.

## Out of scope

- Band resolution, the token grammar, and running a command entry, which belong to
  `epic:running-environments/scope-environment`. The rendering cannot be recorded until a command prints an environment.
