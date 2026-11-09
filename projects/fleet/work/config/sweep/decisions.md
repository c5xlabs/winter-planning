# Sweep decisions

## Where each constant belongs (decision:config/sweep/constant-owners)

**Open.** Which constants belong to the hub, which to the runner, and which to a graph node, and which few must stay
fixed because correctness rests on them; it blocks `product:constants-promoted`.

## A run's configuration recorded (decision:config/sweep/run-config-recorded)

**Open.** Whether the resolved configuration of a run is recorded as a fact, which is what makes comparison across
nights trustworthy and is also a new row on every run; it blocks `product:comparable-run`.

## Reach into the harness adapters (decision:config/sweep/adapter-reach)

**Open.** How far the sweep reaches into the harness adapters, whose own knobs are not blizzard's to redefine; it blocks
`product:constant-inventory`.
