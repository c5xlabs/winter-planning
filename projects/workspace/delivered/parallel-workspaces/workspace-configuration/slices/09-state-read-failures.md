---
name: state-read-failures
depends-on:
  - load-seam
---

# A state file that cannot be read is a configuration error

Makes the read of `.winter/state.toml` fail where
`decision:parallel-workspaces/workspace-configuration/state-read-failures` says it fails, as the state block of
[lock-and-state.md](../spec/lock-and-state.md) states it, and records how the reference tool reads the edges of that
file. Serves `product:configuration-state-ownership`.

## Change

- `crates/core/src/state_registry.rs`: `read` returns `Result<BTreeMap<String, i64>, ConfigError>`. Nothing at the path
  is an empty registry; a read that fails is `FileRead` with the `Io` cause and the path of the file; bytes that are not
  UTF-8 are `FileRead` with an `Io` cause of the kind "invalid data" that carries the decoding error. UTF-8 text that
  does not parse as TOML is `FileRead` with the `Toml` cause and the path, whatever the parser's reason, as the state
  block states it. Nothing of the parser's error is read to decide: no message, no position. The comment that says the
  read never fails is corrected.
- A boolean under `[env_index]` is read as the spec states: `true` as the index 1, `false` as the index 0. Every other
  value that is not an integer stays ignored.
- `crates/core/src/index_lookup.rs` passes the error of `read` on.
- Existing tests whose expectation changes, each renamed for what it asserts:
  - `state_registry.rs`, `a_file_that_cannot_be_read_is_an_empty_registry`: asserts `FileRead` with the `Io` cause and
    the path of the file.
  - `state_registry.rs`, `a_file_that_is_not_valid_toml_is_an_empty_registry`: its text `[env_index`, `alpha = 1`
    asserts `FileRead` with the `Toml` cause and the path, and its bytes `0xff 0xfe` move to a test that asserts
    `FileRead` with an `Io` cause of the kind "invalid data".
  - `state_registry.rs`, `an_entry_that_is_not_an_integer_is_left_out`, whose text holds `flag = true`: asserts the
    entries `kept` with 7 and `flag` with 1, the string, the float, and the array left out.
  - `crates/core/tests/ws_index.rs`, `a_state_file_that_is_not_valid_toml_is_an_empty_registry`, whose state file is
    `[env_index`, `feature-x = 15`: asserts the `Config` ending, exit code 1, and the path of the state file, and still
    asserts that the workspace is left unchanged.
  - `crates/core/tests/ws_index.rs`, `a_state_file_that_cannot_be_read_is_an_empty_registry`, whose state path is a
    directory: asserts the `Config` ending, exit code 1, and the path of the state file.
- The helper `registry_of` of `state_registry.rs` returns the result of `read`; every other test of that file keeps its
  assertion on the registry. No other test under `crates/` hands the reader a state file that does not parse, and no
  recorded scenario under `scenarios/` holds one: the eight fixtures with a `.winter/state.toml` all parse, so no
  existing record differs.
- Tests added to `crates/core/tests/ws_index.rs`: a state file that is not UTF-8, and one that names an environment
  twice in `[env_index]`; each asserts the `Config` ending, exit code 1, and the path of the state file.
- Recorded scenarios, each by `cargo dev record <id> --reference <path>` over a fixture workspace in a temporary
  directory, each asking for `feature-x`, whose suggested index is 40. A plain scenario is replayed against the record.
  A deviation scenario is recorded, then given
  `[deviation] decision = "decision:parallel-workspaces/workspace-configuration/state-read-failures"` and an `expected/`
  tree written by hand that holds exit code 1 and the empty answer a failing `--json` run stores, and recorded again
  with `--rerecord`:

  | Scenario                                 | `state.toml`                                                     | Expected in the record | Form      |
  | ---------------------------------------- | ---------------------------------------------------------------- | ---------------------- | --------- |
  | `ws-index/state-not-utf8`                | bytes that are not UTF-8                                         | exit 1                 | plain     |
  | `ws-index/state-is-directory`            | a directory, the fixture holding `files/.winter/state.toml/keep` | exit 1                 | plain     |
  | `ws-index/state-invalid-toml`            | `[env_index` and `feature-x = 15`                                | exit 0, index 40       | deviation |
  | `ws-index/state-duplicate-key`           | `[env_index]`, `feature-x = 14`, `feature-x = 15`                | exit 1                 | plain     |
  | `ws-index/state-duplicate-inline-key`    | `env_index = { feature-x = 14, feature-x = 15 }`                 | exit 1                 | plain     |
  | `ws-index/state-duplicate-top-level-key` | `a = 1`, `a = 2`, `[env_index]`, `feature-x = 15`                | exit 0, index 40       | deviation |
  | `ws-index/state-duplicate-table-header`  | `[env_index]`, `feature-x = 15`, `[env_index]`, `feature-x = 16` | exit 0, index 40       | deviation |
  | `ws-index/registry-boolean-value`        | `[env_index]`, `feature-x = true`                                | exit 0, index 1        | plain     |
  | `ws-index/state-subtable-over-value`     | `[env_index]`, `feature-x = 15`, `[env_index.feature-x]`         | exit 1                 | plain     |
  | `ws-index/state-dotted-key-over-value`   | `[env_index]`, `a = 1`, `a.b = 2`                                | exit 1                 | plain     |
  | `ws-index/state-duplicate-dotted-key`    | `[env_index]`, `a.b = 1`, `a.b = 2`                              | exit 1                 | plain     |

- Files touched: `crates/core/src/state_registry.rs`, `crates/core/src/index_lookup.rs`;
  `crates/core/tests/ws_index.rs`; `scenarios/ws-index/**` for the eleven scenarios.

## Acceptance

- Unit tests of `state_registry.rs` assert every case of the state block: empty for nothing at the path; `FileRead` with
  `Io` and the path for a failed read; `FileRead` with `Io` of the kind "invalid data" for bytes that are not UTF-8;
  `FileRead` with `Toml` and the path for the text of each of the eight scenarios whose state file is a UTF-8 text that
  does not parse, `state-invalid-toml`, `state-duplicate-key`, `state-duplicate-inline-key`,
  `state-duplicate-top-level-key`, `state-duplicate-table-header`, `state-subtable-over-value`,
  `state-dotted-key-over-value`, and `state-duplicate-dotted-key`; the index 1 for `true` and 0 for `false`. No test
  asserts the text of the parser's error.
- The four component tests of `crates/core/tests/ws_index.rs` named above assert the `Config` ending, exit code 1, and
  the path of the state file.
- `cargo scenarios ws-index` passes: the eight plain scenarios against the record of the reference tool with no
  deviation, the three deviation scenarios against their `expected/` trees. The change reports the eleven records.
- The record of each deviation scenario shows the reference tool exiting 0 with the index 40. If one shows exit code 1,
  that scenario needs no deviation and is kept as a plain one.
- The five plain scenarios whose state file is a text that does not parse are expected to show the reference tool
  exiting 1. One whose record shows exit code 0 becomes a deviation scenario of the same decision, with an `expected/`
  tree as above, and the change reports it as a further text the reference tool reads as empty. The rule does not
  change.
- For `state-not-utf8`, `state-is-directory`, and `registry-boolean-value`, a record that differs from the table stops
  the change, which reports the record and does not change the rule to fit.
- `cargo dev gate slice` passes; no file outside those named changes.

## Out of scope

- The state file writes and the inline table of
  `decision:parallel-workspaces/workspace-configuration/inline-registry-table`, which belong to
  `epic:parallel-workspaces/environment-lifecycle`.
