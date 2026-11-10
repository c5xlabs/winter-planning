# Vision

## The fleet we want

Picture a development team whose junior members never sleep. One or more coding agents — Claude Code today, OpenCode and
Codex later — run continuously on local machines, pulling work from a shared queue, executing each unit in its own
isolated environment, and delivering the results: merged to the main branch in the baseline, or parked at a human-review
gate where one is configured. A human hears from the fleet only when an agent is genuinely blocked.

The fleet keeps working overnight and unattended. It recovers cleanly from crashes and reboots. When an agent gets
stuck, a human steps into that agent's full context in one motion — and steps back out just as easily. And it scales the
way a career does: from a single developer running a handful of agents on one machine to a small team coordinating
across several.

## Autonomous baseline, optional human gates

Work moves through a **user-defined workflow graph** — nodes like build, review, deliver, with cycles for the fix loop.
In the baseline graph, no human is involved: agents build the work, agents review and e2e-test it, and the deliver node
— hub-executed, deterministic — **merges to the main branch**. Where the graph configures it, deliver instead **creates
a pull request** and resolves when that PR merges — merged by a human at the gate, or merged by the hub when the
workflow indicates it should land. Peer-reviewed code as a gate is an **option, not a requirement** — the baseline never
assumes a human is involved.

A human gate is opt-in configuration with two levers: a gate node inserted into the graph ("a human signs off before the
chunk advances"), or a runner's own configuration imposing a gate on any node by node name. That is how trust is tuned:
`persona:senior-se` starts with gates at every station and removes them as confidence grows;
`persona:application-architect` runs the gateless default and steps in only on escalation; both levers are
configuration, never code — `persona:harness-engineer`'s domain.

## What the suite's personas need from blizzard

The three suite personas are described across the whole suite in the program charter. This is what each needs from
blizzard in particular.

### The application architect

Evenings and weekends are when the fleet of `persona:application-architect` earns its keep. They file issues during the
day as ideas arrive, queue work before stepping away, and expect the morning to greet them with landed changes — merged
to main, or waiting at exactly the gates they chose — and, where something went wrong, an escalation precise enough to
act on immediately. What they never expect to find is a wedged fleet or a mystery.

From blizzard they need:

- **Unattended throughput** — file issues, walk away, return to landed work.
- **Truthful status at a glance** — the board never says something the system cannot back.
- **Cheap takeover** — one pasted resume command lands them in the stuck agent's full session context, not in a cold
  reconstruction of it.
- **Zero duplicated or clobbered work** — a structural guarantee, not a probabilistic one.
- **Polyrepo end to end** — their unit of work spans repositories, and the platform treats that as normal.

In short: the [mission](./mission.md)'s success criteria are this persona's needs, written down. Everything in the core
platform serves them — the runner's store, the supervisor loop and its crash recovery, the harness adapters and their
takeover flows, the board.

### The harness engineer

`persona:harness-engineer` is the fleet's mechanic. In blizzard their tuning is lease TTLs, heartbeat intervals, retry
caps, tick rates, model and harness routing — they change one thing, run the fleet, and read the results. From blizzard
they need:

- **A configuration-based system.** Every operational constant lives in config, changeable without touching code. A
  constant buried in code is a dial they cannot turn.
- **Cheap forensics.** The system records what actually happened — facts, not self-reported statuses that can lie — so
  an incident's lifecycle is reconstructable by querying it. (The agent's *reasoning* lives in transcripts on the runner
  machine; the facts cover the lifecycle.)
- **Comparable runs.** A parameter change is evaluated against real outcome data, never against vibes.

What serves them is the facts-only schema, `blizzard runner selftest` and pinned harness versions, and the planner's
shadow-mode comparison discipline.

### The senior software engineer

`persona:senior-se` needs the fleet to earn their trust gate by gate. They want to hand work to the fleet the way they
would hand it to a new team member, and the fleet's structure substitutes for orchestration skill they haven't built
yet. Work arrives already shaped, so deciding *how* something gets executed is not their burden. Output arrives in
trustable units — per-task verdicts, small blast radius. And when the fleet needs them, the escalation is a pasteable
command and the agent's full session context, not a puzzle.

Above all, they need the dial. They start with a human gate at every station — in the workflow graph, or imposed by
their own runner on any node by name — and remove gates one at a time as confidence grows. Blizzard is their on-ramp
from HITL to HOTL, and it builds their confidence the only way that lasts: by being structurally unable to lie about
status or slip work past a gate they configured. Trust is earned gate by gate, never demanded as an entry fee.

What serves them is fail-safe verdicts from the supervisor, the configurable human gates, the planner's routing
doctrine, and the board.

## Why winter is the key binding

Blizzard itself is workspace-agnostic — the workspace is a seam, like the forge and the harness
([mission](./mission.md)). But an orchestrator is only as capable as the chunks of work its agents can safely hold, and
that is why winter matters so much as the reference binding.

Winter is what makes a single agent powerful. A winter feature environment composes one git worktree per project
repository, all on a shared branch, with its own ports and its own running services. That is the machinery behind the
flexible work chunk: one agent takes a chunk of one or more tasks, fans out subagents across one or more feature
environments, and touches one or more repositories — with isolation guaranteed at every level. Two agents on different
chunks never share a working tree, never collide on ports, never trip over each other's services. There is nothing to
lock and nothing to serialize.

The rest of the ecosystem cannot express this. Off-the-shelf orchestrators assume one repository, one worktree, one
branch, one agent — the rigid shape the mission forbids. Winter is what makes the many-to-many chunk executable at all:
not a foundation blizzard is welded to, but the workspace that gives blizzard's agents the most room to work.

## Deterministic shell, intelligent core

One design principle runs through everything blizzard is:

> All loop, arbitration, and recovery logic is boring deterministic code. LLMs supply judgment only at well-defined
> points.

The control plane — the queue, the lease protocol, the reconciliation loop, the fencing, the crash recovery — is
ordinary deterministic software: testable, `kill -9` safe, and never dependent on a model behaving well. Models are
invited in only where judgment is genuinely the job — writing the code, reviewing and testing it, optionally planning
how open work should be shaped into batches, and rendering the verdict a finished worker hands back.

The reason is easy to hold: an LLM can be wrong in judgment, and the system is built to survive that. It is never handed
a lever that lets it be wrong in arithmetic.
