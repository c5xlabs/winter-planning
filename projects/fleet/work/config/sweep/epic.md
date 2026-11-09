---
epic: config/sweep
refinement: outline
status: planned
---

# Sweep

Part of `initiative:config`; its product requirements are below.

## The problem

Every operational constant beyond the model a session runs on is still a number somebody chose once in code, out of
reach of the harness engineer who wants to move it and see what it cost. A worker's environment is scattered the same
way: tuning how a worker behaves means setup spread across systemd, a shell profile, and the runner config.

## The outcome

The rest of the configuration surface `initiative:config` promises is opened: every operational constant beyond the
model a session runs on is inventoried and promoted under one resolution order, and a run's values can be recovered
after it, so two runs of the same chunk can be compared.

## Requirements

### The inventory (product:constant-inventory)

A deliberate sweep finds the operational constants across hub, runner, and graph execution, each one named and located.
The sweep is the work; the plumbing after it is mechanical.

### Every constant promoted (product:constants-promoted)

Each constant the sweep finds becomes configuration, with its current value as its declared default and a written
account of what moving it does, so a reader can tell a tuning knob from a load-bearing invariant.

### One resolution order (product:one-resolution-order)

Where a value may be stated in more than one place, the precedence is declared once and holds everywhere, so the answer
to "what is this fleet actually running with" is a lookup rather than an investigation.

### A worker's environment in the runner's config (product:worker-env-in-runner-config)

Environment variables a worker should see are set in the runner's toml or an env file it names. Today
`worker_env_passthrough` only passes through what is already set outside the runner, so tuning worker behavior, test
parallelism through `PYTEST_XDIST_AUTO_NUM_WORKERS` for instance, means scattered setup across systemd, a shell profile,
and the runner config.

### The comparable run (product:comparable-run)

The harness engineer changes one constant, runs the same chunk again, and can attribute the difference, which means the
values a run used are recoverable after the fact, not only before it.
