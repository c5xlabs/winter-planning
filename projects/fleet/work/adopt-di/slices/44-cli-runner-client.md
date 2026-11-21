---
name: cli-runner-client
depends-on:
  - cli-worker
  - epic:config/runtime
code:
  - blizzard/src/blizzard/runner/cli/daemon.py::RunnerDaemon
  - blizzard/src/blizzard/runner/cli/transcript.py::transcript_list
  - blizzard/src/blizzard/runner/cli/transcript.py::transcript_get
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_clear
---

# Runner daemon-client command containers

Delivers the runner daemon-client boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports` and `engineering:born-in-container` under `decision:adopt-di/declared-cutover`.

## Change

- Migrate the runner verbs that only reach a running host through `RunnerDaemon` to client registrations: control
  (status, pause, start, clear-floor, takeover, requeue, selftest), settings (including apply and export), traces,
  `transcript list` and `get`, `prompt clear`, and the workspace and subscription record verbs.
- Load no store, composition or daemon provider module for these verbs; keep `RunnerDaemon`'s socket and override
  selection, and the `flag_gate` helper the verbs share.
- The local verbs in the same groups (`transcript backfill` and `reship`, `prompt list`, `get`, `install`, `diff` and
  `status`) migrate in `cli-runner-writes` and `cli-runner-local`; a group loads only its selected command's
  registrations.
- Split the modules whose top-level imports pull the daemon stack into these verbs: `runner/cli/transcript.py` imports
  `LoopWiring` at module level, which imports `runner.composition`, and `runner/cli/prompt.py` imports
  `runner.cli.runtime`. Move the local verbs' bodies into their own modules, or defer those imports into the local
  verbs, so loading `transcript list`, `get` or `prompt clear` imports neither.

## Acceptance

- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) show these verbs import
  no store, app or composition module, including when their group's local siblings are registered.
- `blizzard:component-test` (`tests/component/runner/cli`): carrier fakes control the daemon client through the
  container; outputs, refusals and exit codes are unchanged.
- Each graph validates; `blizzard:typecheck` and `blizzard:cli-contract` (the command surface) pass.
