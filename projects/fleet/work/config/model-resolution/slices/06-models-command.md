---
name: models-command
depends-on:
  - report-builder
code:
  - blizzard/src/blizzard/runner/cli/__init__.py::_COMMANDS
---

# Models command

Registers `blizzard runner models` and its file path: loading the configuration without a daemon, the terminal
rendering, `--json`, and the exit codes ([command.md](../spec/command.md) §Invocation, §Exit status, §Terminal
rendering). Delivers `product:terminal-and-json` and the terminal side of every other `product:` requirement of the
epic. Shaped by `decision:config/model-resolution/plain-terminal-output` and
`decision:config/model-resolution/findings-exit-zero`. Waits on the open questions of `--runner` and the config file
name.

## Change

- `runner models` builds the report in the CLI process, with every harness `not-probed`, when no daemon is reachable or
  `--from-file` is given.
- It renders the four sections and their `!` lines, or prints the JSON document alone.
- It exits `0`, `1`, `2`, or `3` as the spec's table gives.

## Acceptance

- The terminal table and the JSON form name the same layers and the same effective values for the same configuration.
- The sample configuration renders the [worked sample](../../assets/model-resolution.txt) layout within 120 columns.
- `--graph` with `--chunk` exits `2`; `--chunk` with the hub unreachable exits `3` and still prints the tables.
- The command writes nothing to the configuration, the runner store, or the hub.
