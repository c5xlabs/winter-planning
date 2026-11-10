# Model resolution decisions

## The daemon builds the report (decision:config/model-resolution/daemon-built-report)

**Decider:** engineering

When a daemon is running, it builds the report from the configuration, bindings, and harness health it loaded at
startup, because those are what its next spawn uses; the configuration file is read only with `--from-file` or with no
daemon.

## Findings never fail the command (decision:config/model-resolution/findings-exit-zero)

**Decider:** engineering

A printed report exits `0` whatever it finds, because the command reports and does not judge.

## Plain terminal output (decision:config/model-resolution/plain-terminal-output)

**Decider:** engineering

The terminal form is plain `click.echo` with fixed-width columns and no color, matching the rest of the runner CLI.
