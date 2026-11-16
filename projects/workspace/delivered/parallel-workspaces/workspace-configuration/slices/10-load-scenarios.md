---
name: load-scenarios
depends-on:
  - load-seam
---

# The load is recorded against the reference tool

Records what the change named load-seam built: the two files, the merge, the checks made in each file, and the three
keys `settings.rs` reads, through `ws index`. Adds the deviation scenarios of
`decision:parallel-workspaces/workspace-configuration/repository-array-type` and
`decision:parallel-workspaces/workspace-configuration/element-type-validation` that the load itself decides. Serves
`product:effective-configuration` and `product:configuration-state-ownership`.

## Change

- Recorded scenarios, each plain, by `cargo dev record <id> --reference <path>` over a fixture workspace in a temporary
  directory: `ws-index/local-overrides-aliases`, `ws-index/local-replaces-envs-per-workspace`,
  `ws-index/local-path-is-directory`, `ws-index/invalid-toml-local`, `ws-index/config-not-utf8`,
  `ws-index/merge-shared-git-not-table`, `ws-index/agent-overrides-not-table-local`,
  `ws-index/invalid-adopt-extensions`, and `ws-index/main-branch-number`, whose shared file holds `main_branch = 7`.
- Deviation scenarios. Each is recorded, then given its `[deviation] decision = "…"` and an `expected/` tree written by
  hand that holds exit code 1 and the empty answer a failing `--json` run stores, in the form `repo-remove/not-found`
  stores them, and recorded again with `--rerecord`:
  - under `decision:parallel-workspaces/workspace-configuration/repository-array-type`:
    `ws-index/repository-key-not-array`, whose shared file holds `project_repository = "abc"`, and
    `ws-index/local-repository-key-not-array`, whose local file holds `standalone_repository = "abc"` beside a shared
    array;
  - under `decision:parallel-workspaces/workspace-configuration/element-type-validation`:
    `ws-index/git-excludes-string`, whose shared file holds `git_excludes = "abc"`, and
    `ws-index/shared-table-key-array-of-pairs`, whose shared file holds `keybindings = [["leader", "x"]]` and whose
    local file holds `[keybindings]` with `timeoutlen = 500`.
- The probe `ws-index/config-integer-beyond-64-bits`, whose shared file holds an unknown key with the value
  `123456789012345678901234567890`. Where the binary's parser accepts the text, it is a plain scenario; where it
  refuses, it is a deviation scenario under `decision:parallel-workspaces/workspace-configuration/toml-dialect` with an
  `expected/` tree as above.
- Files touched: `scenarios/ws-index/**` for the fourteen scenarios.

## Acceptance

- `cargo scenarios ws-index` passes: the nine plain scenarios against the record of the reference tool with no
  deviation, the four deviation scenarios against their `expected/` trees, and the probe as its case above says.
- The record of `invalid-toml-local`, of `config-not-utf8`, and of `main-branch-number` shows the reference tool
  exiting 1.
- The record of the four deviation scenarios shows the reference tool exiting 0; the change reports the four records. If
  one shows exit code 1, that scenario needs no deviation and is kept as a plain one.
- The record of the probe shows the reference tool exiting 0 with the index; the change reports the record and which of
  the two forms the scenario took. If the record shows exit code 1, the change stops and reports it.
- No file under `crates/` changes. If a plain scenario fails against the binary, the change stops and reports it.

## Out of scope

- Scenarios of keys a later change reads.
