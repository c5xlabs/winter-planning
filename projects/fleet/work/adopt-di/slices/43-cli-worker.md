---
name: cli-worker
depends-on:
  - cli-operator
  - epic:config/runtime
code:
  - blizzard/src/blizzard/runner/cli/__init__.py::_RunnerGroup
  - blizzard/src/blizzard/runner/cli/worker_call.py::WorkerSession
---

# Runner worker-command containers

Delivers the runner worker-verb boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports` and `engineering:born-in-container` under `decision:adopt-di/declared-cutover`.

## Change

- Migrate the worker verbs, every verb calling through `WorkerCall`: `heartbeat`, `session-end`, `ask`, `attach`,
  `work-items`, `pm-items`, `chunk`, `artifact`, `garden`, `finding`, `scope`, `analytics` and `opencode drive`.
  `opencode attend` calls no `WorkerCall` and migrates with the local verbs in `cli-runner-local`.
- `_RunnerGroup`'s `WorkerSession` stays the group's: it opens before the selected verb's container, which receives it
  as a borrowed context value ([Entrypoint resolution](../spec/entrypoints.md#interfaces)).
- Register worker-session, launcher, loopback/OpenCode transport, HTTP and trace dependencies in their own areas.
- Keep `_RunnerGroup` from importing/registering daemon runtime siblings; retain selected-command loading.
- Use command carrier inputs, remove replaced constructors and update worker wiring exemplars.

## Acceptance

- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) show worker verbs never
  import `runner.cli.runtime`, app/composition/store stacks, or daemon provider modules.
- `blizzard:component-test` (`tests/component/runner/cli`): carrier fakes control HTTP, launcher and transports through
  the container without module patching.
- Each worker graph validates; command outputs/refusals, `blizzard:typecheck` and `blizzard:cli-contract` (the command
  surface) pass.
