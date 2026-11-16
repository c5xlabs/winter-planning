---
name: configuration-proof
depends-on:
  - repository-entries
  - settings-keys
  - env-bands
  - model-tiers-and-agent-overrides
  - repo-remove-loads-configuration
---

# The whole model, proven through the two commands

Proves the outcome of the epic: `ws index` and `repo remove` run against a configuration that sets every key, and the
cross-key rules that need two children hold. Serves `product:effective-configuration` and
`engineering:typed-configuration-failures`.

## Change

- Recorded scenarios, each plain, by `cargo dev record <id> --reference <path>` over a fixture workspace in a temporary
  directory: `ws-index/full-configuration`, whose two files set every key of
  [configuration-model.md](../spec/configuration-model.md) validly, with every merge rule in use;
  `ws-index/prefix-collision`; `ws-index/unknown-keys-ignored`; `ws-index/nested-workspace-root`, run from inside a
  nested workspace of the fixture; `repo-remove/full-configuration`.
- A unit test in `crates/core/src/config_load.rs` puts the two configuration files of `ws-index/full-configuration`,
  taken into the test at compile time with `include_str!`, into `MemoryFiles`, calls `load`, and asserts every field of
  the `WorkspaceConfig`. It reads no file when it runs and uses no `OsFiles`. It is a unit test because no public
  function returns a `WorkspaceConfig`.
- A component test in the new file `crates/core/tests/configuration.rs` runs `ws index` over a workspace whose `prefix`
  equals a standalone's and asserts the `PrefixCollision` ending.
- The built binary writes configuration files in `repo-remove/full-configuration`. It is run only against fixture
  workspaces in temporary directories, never against a real workspace.
- Files touched: `crates/core/src/config_load.rs`, tests only; `crates/core/tests/configuration.rs`;
  `scenarios/ws-index/**` and `scenarios/repo-remove/**` for the five scenarios.

## Acceptance

- `cargo scenarios ws-index` and `cargo scenarios repo-remove` pass, the five new scenarios against the record of the
  reference tool with no deviation.
- The unit test builds the expected `WorkspaceConfig` as a struct literal with no `..`, so a field added later without a
  value in the fixture fails to compile.
- The component test asserts that `PrefixCollision` carries the prefix and the standalone's name.
- `cargo dev gate slice` and the full gate pass; no code outside tests changes.
