---
name: cli-runner-writes
external: github:paul-gross/blizzard#2501
depends-on:
  - cli-runner-dual-mode
code:
  - blizzard/src/blizzard/runner/cli/store.py::reset
  - blizzard/src/blizzard/runner/cli/runners.py::ingest
  - blizzard/src/blizzard/runner/composition.py::build_runner_ingest
  - blizzard/src/blizzard/runner/cli/transcript.py::transcript_reship
  - blizzard/src/blizzard/runner/cli/transcript.py::transcript_backfill
---

# Runner local store-write command containers

Delivers the runner local store-write verbs in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports` and `engineering:ordered-teardown`.

## Change

- Migrate `store reset`, `ingest`, `transcript reship` and `transcript backfill` to command roots over the selected
  store graph, under `store_write_guard`; `reship` and `backfill` keep refusing while a host runs.
- Register the ingest graph (`build_runner_ingest` with `HostKeyCiphers` and `DotenvTokenFiles`) in its own area,
  resolved only by `ingest`'s command root.
- The transcript verbs resolve the loop collaborators `runner-loop` registered in place of `LoopWiring.with_context`.
- Retire the compatibility entrypoints whose last callers these verbs are: `build_runner_ingest` (`ingest`), and
  `build_runner_store_files` with the `build_runner_transfer` and `build_runner_removal` it composes (`store reset` and
  `ingest`; its `host_roster` callers moved onto the command root in `cli-runner-dual-mode` and `cli-runner-host`), and
  `build_read_stores`, whose last callers are `RunnerStoreFiles.held_bindings` and `RunnerBoundStoreFiles.runner_view`
  inside `build_runner_store_files`.

## Acceptance

- `blizzard:component-test` (`tests/component/runner/cli`): each verb keeps its output and exit status, including
  `transcript reship`'s refusal while a host runs; a failed verb closes its command root.
- `blizzard:component-test`: `ingest` imports declared runners and tokens as today; `backfill` reuses the selected
  runner's loop collaborators and closes what it owns once.
- `blizzard:typecheck` and `blizzard:cli-contract` (the command surface) pass unchanged.
