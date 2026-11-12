---
epic: config/runtime
refinement: refined
status: in-progress
external: github:paul-gross/blizzard#2124
---

# Runtime

Part of `initiative:config`; its product requirements are in [requirements.md](./requirements.md).

## The problem

`persona:harness-engineer` tunes a fleet the way a mechanic tunes an engine: move one thing, listen, move it back. Today
every one of those moves on a runner costs a restart of the host it lives on, and that restart is not free. It
interrupts every session each of the host's runners is carrying, each of which has to find its way back to where it was
cut off, so the engineer who wants to lower a budget at ten at night, or hold a runner to fewer agents while they watch
a flaky chunk, either waits for the whole host to go idle or pays for the experiment in broken sessions. Adding a runner
to a host is the same: the new runner waits for the host's next start.

The hub is kinder only by degree. Its work sources, repositories, and secrets already change while it runs, since
`epic:live-config`, but the settings that govern how it behaves (how often its background passes run, how much
transcript it will accept, how hard it throttles a login) are read once when it starts and held until it stops.

Nothing about a runner needs it to be this way. The host reads its file once when it starts and freezes what it read
into the objects it builds, and no path was ever built to change them afterwards. It freezes that reading into the
context each runner's loop caches on its first tick, into the timer each loop runs on, into each runner's harness
adapters and usage samplers, and into the set of runners it starts. Nothing it does afterwards looks at the file again,
the hub sends it back nothing but the fleet-wide pause, and the loop's own tick is not even in the file: it is an
environment variable read at boot.

## The outcome

The operational settings of both daemons become something an operator changes while they run, for the runner first and
the hub beside it, and the moment a change takes hold becomes something the operator can predict rather than discover.

This epic is the companion of `epic:config/sweep`. The sweep decides *what* is configurable: it finds every constant
somebody chose once and promotes it, with an account of what moving it does. This epic decides *when* a change reaches
the running fleet. Neither depends on the other to land, but each is worth more beside the other: a constant the sweep
promotes is only half a knob if turning it means a restart, and a setting this epic makes live is only as useful as the
list of settings that exist.

The two daemons are different machines with different owners, different failure modes, and different things at stake in
a restart, so the epic has a runner half and a hub half, and each daemon owns its own settings outright. The hub's
settings are changed through the hub's API; a runner's are changed through that runner's API, and stay on the runner.
Neither daemon holds the other's, and neither file is a way to change a running daemon. The runner half is where a
restart hurts most; the hub half follows it, and the epic needs both.

## Left for later

- The settings revision a session ran under stays on the runner. Recording it as a fact on the hub, which comparing runs
  across a fleet would want, is left for `epic:config/sweep` to ask for.
- Having a runner take its side of the settings it shares with the hub (the hub's offline cutoff, its usage staleness,
  and its transcript record cap) from the hub's own settings is left for a later epic of `initiative:config`.

## What this epic is not

It does not decide which constants become settings; that is `epic:config/sweep`, and every setting the sweep promotes
arrives declaring its lifetime. It does not let either daemon hold or change the other's settings: the hub does not tune
a runner, and a runner does not tune the hub. It does not reach into the harnesses' own configuration, which is theirs.

A `boot` setting changed without a restart, a workspace whose disk blizzard reconciles, and a check between two settings
inside one daemon whose sides are both still constants all wait for a reason to build them. Narrowing the runner's
human-lane writes that predate this epic to a role belongs to the runner's auth, not to this epic.
