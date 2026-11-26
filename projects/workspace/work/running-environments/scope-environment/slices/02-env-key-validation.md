---
name: env-key-validation
depends-on:
  - module-scaffold
code:
  - winter-workspace/crates/core/src/config_model/env_bands.rs::env_value
---

# Band keys that are not shell names are refused at load

Adds the key check of [resolution.md](../spec/resolution.md), Key validation at load: a declared key of any band that
does not match `^[A-Za-z_][A-Za-z0-9_]*$` is `ConfigError::EnvVar` with `EnvVarFault::InvalidKey`, checked first, for a
literal and a command entry alike. Delivers the load bullet of `product:layered-runtime-values`, a registered deviation
under `decision:running-environments/scope-environment/resolution-compatibility`, because the reference emitted an
`export` line that broke providers' `eval`. It extends the `EnvVarFault` of the frozen
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
whose other checks stand.

## Change

- `config_model/env_bands.rs` refuses an invalid key before any other fault of its entry, in the workspace, feature, and
  named bands.
- Every command that loads the workspace refuses such a configuration with exit 1, as any `ConfigError` is.

## Acceptance

- Unit tests: `A-B`, `1A`, `A.B`, an empty key, and a non-ASCII letter are each `InvalidKey` naming band and key; `_A`
  and `a1` pass.
- A unit test: a command entry under an invalid key that also has an unknown field answers `InvalidKey`.
- A component test through `InProcess`: a workspace whose `env.feature.vars` declares `A-B` fails `ws index alpha` with
  the `ConfigError`, exit 1.
- The six `ws-index` env-band scenarios of `epic:parallel-workspaces/workspace-configuration` still pass.
- `cargo dev gate slice` exits 0.
