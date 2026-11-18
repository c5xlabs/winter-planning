---
name: top-level-refusals
depends-on:
  - layered-load
code:
  - winter-workspace/crates/core/src/config.rs
  - winter-workspace/crates/core/src/termination.rs::ConfigError
---

# Refusals of the top-level keys and the dashboard layout

Adds to the load the refusals of Top-level keys, Value types for the top-level keys, Removed keys for
`service_orchestrator` and `session_prefix`, and the `tui.dashboard.layout` row of Tables in
[configuration-model.md](../spec/configuration-model.md), each at its place in the Load order. Delivers the
operation-relevant, unknown-key, and removed-key bullets of `product:configuration-validation`, under
`decision:parallel-workspaces/workspace-configuration/validation-boundary` and the registered deviation
`decision:parallel-workspaces/workspace-configuration/element-type-validation`.

## Change

- `ConfigError` gains `RemovedKey`, `InvalidChoice`, and `ValueType`, with `RemovedKey`, `ChoiceKey`, `TypedKey`,
  `ConfigValue`, and `ValueType` as the spec's Domain gives them. Each message names the key, and a removed key's
  message names its replacement from the Removed keys table.
- The load refuses, in Load order: an invalid `tui.dashboard.layout` (step 2); `service_orchestrator`, then
  `session_prefix`, present in the merged document (step 4); an `adopt_extensions` that is not `none`, `winter`, or
  `all` (step 5). The top-level `main_branch` is checked after the removed keys of step 4, and the top-level
  `git_excludes` at step 9.
- Top-level `main_branch`: a non-string is `ValueType` unless it is one of the empty forms the key row lists, which read
  as the default. Top-level `git_excludes`: anything but an array of strings is `ValueType`; an absent key and an empty
  array are accepted.
- Unknown top-level keys and unknown keys of open tables are ignored with no message.
- No value is kept: `ws index` still reads only its two keys.

## Acceptance

- Unit tests, one per refusal: each of `service_orchestrator = "tmux"` and `session_prefix = "x"` in the shared file,
  and each in the local file only, is `RemovedKey` naming the replacement; `adopt_extensions = "some"` is
  `InvalidChoice`; `[tui.dashboard] layout = "grid"` is `InvalidChoice`; `main_branch = 3` is `ValueType`, while
  `main_branch = ""`, `false`, `0`, `[]`, and `{}` load; `git_excludes = "abc"`, `git_excludes = {}`, and
  `git_excludes = ["a", 1]` are `ValueType`, while an absent key and `git_excludes = []` load.
- Unit tests of order: `session_prefix` with an invalid `adopt_extensions` reports `RemovedKey`; `main_branch = 3` with
  an invalid `adopt_extensions` reports `ValueType`; an invalid `adopt_extensions` with `git_excludes = "abc"` reports
  `InvalidChoice`.
- A unit test with an unknown top-level key and an unknown key under `[git]` loads with no message.
- A component test of `ws index` with `session_prefix` set exits 1 with a diagnostic naming `service_prefix`, and writes
  nothing to stdout.
- `cargo dev gate full` exits 0.

## Out of scope

- The repository, env band, and model tier refusals: `repository-entries`, `env-bands`, `model-tiers`.
- The removed `[env.vars]`: `env-bands`, where the load meets it.
