---
name: cli-worker
depends-on:
  - cli-operator
---

# Runner worker-command containers

Delivers the runner worker-verb boundary in spec [Entrypoint resolution](../spec/entrypoints.md), serving
`engineering:selective-command-imports` and `engineering:born-in-container` under `decision:adopt-di/declared-cutover`.

## Change

- Register worker-session, launcher, loopback/OpenCode transport, HTTP and trace dependencies in their own areas.
- Keep `_RunnerGroup` from importing/registering daemon runtime siblings; retain selected-command loading.
- Use command carrier inputs, remove replaced constructors and update worker wiring exemplars.

## Acceptance

- Fresh-interpreter loaded-module tests show worker verbs never import `runner.cli.runtime`, app/composition/store
  stacks, or daemon provider modules.
- Carrier fakes control HTTP, launcher and transports through the container without module patching.
- Each worker graph validates; command outputs/refusals, strict typing, relevant tests and `blizzard:cli-contract` pass.
