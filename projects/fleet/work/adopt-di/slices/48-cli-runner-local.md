---
name: cli-runner-local
external: github:paul-gross/blizzard#2502
depends-on:
  - cli-runner-host
  - cli-runner-writes
code:
  - blizzard/src/blizzard/runner/cli/runtime.py::read_host_stores
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_list
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_show
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_diff
  - blizzard/src/blizzard/runner/cli/prompt.py::prompt_status
  - blizzard/src/blizzard/runner/cli/harness.py::harness_status
  - blizzard/src/blizzard/runner/cli/external_usage.py::external_usage_probe
  - blizzard/src/blizzard/runner/cli/opencode.py::opencode_compatibility
  - blizzard/src/blizzard/runner/cli/opencode.py::opencode_attend
---

# Runner offline-read command containers and CLI closure

Delivers the remaining runner local verbs in spec [Entrypoint resolution](../spec/entrypoints.md) and retires the runner
CLI's compatibility entrypoints, serving `engineering:selective-command-imports`, `engineering:container-only` and
`engineering:ordered-teardown`.

## Change

- Migrate the offline-read and host-file verbs to command roots: `prompt list`, `get`, `diff` and `status`,
  `harness status`, `external-usage probe`, `opencode compatibility` and `opencode attend`. `prompt status` and `diff`
  read the store offline through the selected read graph in place of `read_host_stores`. `opencode attend` closes its
  command container before it `exec`s the operator's OpenCode.
- Retire `build_host_read_stores` and `read_host_stores`, whose last callers are `prompt status` and `diff`; the record
  verbs, `prompt clear` and `install` (through `_home_record`) and `read_recorded_host` left it in
  `cli-runner-dual-mode`. Every other CLI-edge store-factory compatibility entrypoint `runner-stores` left retired with
  its last caller, in `cli-runner-dual-mode`, `cli-runner-host` or `cli-runner-writes`; `read_stores` was deleted in
  `runner-stores`. A gate confirms none remains.
- Remove the remaining command/manual-store constructor paths; keep local-provider imports inside selected command
  modules and preserve configured binding before offline reads.

## Acceptance

- `blizzard:component-test` (`tests/component/runner/cli`): each verb keeps its output and exit status, with invoker
  overrides and offline binding.
- `blizzard:structural-gate`: no runner CLI module calls a retired compatibility entrypoint, and a planted call fails.
- `blizzard:unit-test` fresh-interpreter loaded-module tests (`tests/support/loaded_modules.py`) keep local providers
  out of worker and daemon-client verbs; all command graphs validate; `blizzard:typecheck`, `blizzard:cli-contract`,
  `blizzard:config-contract` and `blizzard:compat` pass unchanged.
