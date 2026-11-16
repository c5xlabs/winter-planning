---
name: settings-keys
depends-on:
  - load-seam
---

# Top-level settings and the small tables

Fills `config_model/settings.rs` with the keys of [configuration-model.md](../spec/configuration-model.md) that no other
child owns and the file does not read yet: Top-level keys other than the four of `ports.rs` and the three `settings.rs`
already reads (`adopt_extensions`, `git_excludes`, `main_branch`), the `git`, `capabilities`, `keybindings`, `tui`,
`core_checks`, `space`, and `lint` rows of Tables, Removed keys other than `[env.vars]`, and the raw `provision` and
`service`. Serves `product:configuration-validation`.

## Change

- Each key is read with the default, coercion, or refusal of its row; `tui.dashboard.layout` is `InvalidChoice`, as
  `adopt_extensions` already is; `service_orchestrator` and `session_prefix` are `RemovedKey`; `git`, `git.user`, and
  the two identity values follow the last block of Value types.
- A `keybindings`, `tui`, `capabilities`, `core_checks`, or `space` that is not a table, and a `tui.dashboard`,
  `keybindings.bindings`, `core_checks.file_size`, or `space.kinds` that is not a table, reads as absent: the defaults
  of its rows, with no message, as the spec's block under Tables states from the reference source.
- `provision` and `service` are kept raw; a `provision` that is not a table is empty, and a `service` that is not an
  array is empty.
- Recorded scenarios, each plain, by `cargo dev record <id> --reference <path>` over a fixture workspace in a temporary
  directory: `ws-index/removed-session-prefix`, `ws-index/removed-service-orchestrator-local`,
  `ws-index/invalid-dashboard-layout`, `ws-index/git-user-string`, `ws-index/wrong-types-fall-back`; and
  `ws-index/keybindings-is-string`, `ws-index/tui-is-string`, `ws-index/capabilities-is-string`,
  `ws-index/core-checks-is-string`, `ws-index/space-is-string`, each with its key set to a string in the shared file
  alone.
- Files touched: `crates/core/src/config_model/settings.rs`; `scenarios/ws-index/**` for the ten scenarios.

## Acceptance

- Unit tests assert every row of Top-level keys and Tables this child owns: the default, each accepted form, and each
  wrong type with its fallback or its variant.
- A unit test table asserts, for each of the five tables and the four sub-tables named above set to a string, that the
  load succeeds and the fields have their defaults.
- A unit test asserts that a removed key present only in the local file is refused and that the message names its
  replacement.
- `cargo scenarios ws-index` passes, every scenario against the record of the reference tool with no deviation. The
  record of each of the five `-is-string` scenarios shows exit code 0 and the index; if one shows exit code 1, the
  change stops and reports it, and builds no refusal for that key.
- `cargo dev gate slice` passes; no file outside those named changes.

## Out of scope

- `lint.ignore`, which the lint rules read themselves; `[env]`, `[model_tiers]`, `[agent_model_overrides]`.
- The top-level `git_excludes` and `main_branch`, which `settings.rs` already reads.
