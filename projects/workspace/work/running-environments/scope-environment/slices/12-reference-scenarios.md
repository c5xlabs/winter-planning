---
name: reference-scenarios
depends-on:
  - env-key-validation
  - env-command
---

# Recorded scenarios of `winter env`

Records `winter env` against the reference tool and replays it against the binary, proving
[env-command.md](../spec/env-command.md), [resolution.md](../spec/resolution.md), and
[value-lookups.md](../spec/value-lookups.md) end to end under `decision:functional-parity`; each registered deviation
carries its `[deviation]` decision and expected result. It also records the number rendering of `env.*.vars` that
`epic:parallel-workspaces/workspace-configuration` could not record until a command printed an environment.

## Change

- Scenarios under `scenarios/env/`, recorded with `cargo dev record` from the reference tool, each over a fixture
  workspace whose `.winter/state.toml` records its environments; command entries run fixture scripts under the root.
- No fixture declares a service, a provision handler, or an extension.
- No production code changes.

## Acceptance

- Scenarios: the workspace scope; a feature scope with workspace, feature, and named bands; a named band for another
  environment left unread; references with an offset; a masked command entry; `--resolve` of raw, dotenv, and json
  entries with `exports`; an unknown scope, with and without a directory of that name; a failing lookup; integer and
  float literals.
- Deviation scenarios, each with its decision and `expected/`: a recorded index at the buffer slot under
  `decision:running-environments/scope-environment/incomplete-launch-refusal`; a band key `A-B`, a substituted value
  holding a space filling one word, and JSON `true` and `1e16` under
  `decision:running-environments/scope-environment/resolution-compatibility`; a substituted value holding `${X}` and an
  offset over a non-integer under `decision:running-environments/scope-environment/persistent-value-exposure`; a lookup
  run with a stale `WINTER_ENV` in the caller's environment under
  `decision:running-environments/scope-environment/inherited-environment`.
- `cargo dev gate full` exits 0.
