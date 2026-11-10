---
name: daemon-report
depends-on:
  - models-command
code:
  - blizzard/src/blizzard/runner/cli/daemon.py::RunnerDaemon.reach
---

# Daemon report

Adds the daemon's local API read for the report and points `runner models` at it whenever a daemon is reachable
([command.md](../spec/command.md) §Flows). Shaped by `decision:config/model-resolution/daemon-built-report`. Waits on
the open question of the route's path.

## Change

- The daemon builds the report in-process from the configuration, bindings, and harness health it loaded at startup.
- `runner models` reaches the daemon the way `runner status` does and uses the file path only with `--from-file` or no
  daemon.
- A configuration file that differs from what the daemon loaded sets `config_changed_on_disk`, a header line, and a
  `stale-daemon-config` note.
- `openapi/runner.openapi.json` gains the route.

## Acceptance

- With a daemon running, `runner.source` is `daemon` and harness states come from its health cache.
- Editing the configuration file after the daemon starts yields `config_changed_on_disk: true` and the `!` line.
