---
initiative: config
refinement: outline
epics:
  - model-resolution
  - sweep
  - runtime
---

# Config

The numbers a fleet runs by become configuration an operator can read, change, and compare, instead of constants fixed
in code.

## The problem

A young platform is full of numbers somebody chose once. How long a lease lives, how many times a node retries before
escalating, how long the reconciler sleeps between sweeps: each was settled during a build by someone who was not
running this fleet, on this machine, against this repository. They are usually close enough to right, and when one is
wrong it is wrong in code, costing a change, a review, and a release.

`persona:harness-engineer` feels this most sharply, because their whole craft is comparison: run the same chunk twice
with one variable moved and see what it cost. A constant they cannot reach is a question they cannot ask.

## The outcome

The configuration surface is surveyed and opened, a survey-then-sweep effort rather than a large build. One corner of it
is answered early, because there the answer decides what a night costs: which model a session actually runs on.
`epic:config/model-resolution` gives the runner a read-only command that shows how each tier, effort, and session pool
resolves to a model, and which layer supplied each value. `epic:config/sweep` inventories every other operational
constant and promotes it under one resolution order. And a knob is only half a knob if turning it means a restart, so
`epic:config/runtime` makes the surface live as well as open: hub and runner settings an operator changes while the
daemons run, each declaring when a change takes hold.
