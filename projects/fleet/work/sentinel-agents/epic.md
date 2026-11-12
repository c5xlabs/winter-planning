---
epic: sentinel-agents
refinement: outline
status: planned
---

# Sentinel agents

A standalone epic; its requirements are inline below.

## The problem

Every agent the fleet runs today is a worker: one of many, scaled out across runners, each working its own chunk. That
is right for building, and wrong for the work that has to see the factory whole. When a chunk escalates and a human is
needed, nobody looks into why until the operator does; when the fleet stalls, nothing notices until morning. Hand that
oversight to workers and several of them react to the same event at once, each with a partial view, racing one another
over the same chunks. The work wants one agent, holding the role alone, the way a lock is held.

## The outcome

A tenant defines agents that each hold a single identity within a declared scope, and the factory guarantees that no two
sessions ever act in that role at the same time. Something happens, the one agent is woken, reacts, and stops. The first
of them oversees escalations: it looks into why a chunk escalated or is waiting on a human, settles what it can, and
leaves the operator a clear account of what it could not. A sentinel is not one per factory but one per scope, and
within its scope it is alone: a single activation, never two.

## What this epic is not

- A worker by another name. A sentinel agent works no chunk's graph and is never scaled out.
- A hub with a model. The hub never holds model access; the agent runs on a runner like every other session.

## Depends on

- `epic:worker-profiles`, loosely: an agent may run under a profile, though it does not have to.

## Requirements

### Defined by the tenant (product:tenant-defined)

Each tenant configures its own sentinel agents: the role each plays, what it is told to do, and what wakes it. Beyond
the factory's own oversight, a role may be whatever the tenant wants held by one agent, such as granting every new user
a guest role.

Acceptance:

- A tenant defines a sentinel agent, and it runs without any change to blizzard.

### One per scope (product:sentinel-scope)

Each agent declares the scope it is singular within, such as one per tenant or one per project
(`decision:sentinel-agents/scopes`).

Acceptance:

- An agent declared one per project runs as exactly one agent in each project.

### Never two at once (product:exclusive-role)

The factory guarantees that no two sessions act in the same role, in the same scope, at the same time, including while a
runner is lost or a session is handed between runners. The guarantee rests on the turn: only the holder of the agent's
one granted turn may act (`product:turn-at-a-time`).

Acceptance:

- However many runners can run an agent, at most one session acts in its role at any moment.

### Every turn reaches the factory (product:every-turn-recorded)

The factory holds every turn a sentinel takes, as it happens, so what the agent did and why is always traceable. A turn
the factory never received is a turn that did not count.

Acceptance:

- For any act a sentinel made, the operator reads the turn that made it and the turns that led to it.
- A turn a runner took but never delivered changes nothing the factory acts on.

### One turn at a time (product:turn-at-a-time)

A sentinel takes one turn, the runner commits it to the hub, and only then does the hub grant the next. The agent is
never more than one turn ahead of what the factory holds, and a runner that falls silent mid-turn loses its grant
rather than holding the role (`decision:sentinel-agents/turn-timeout`).

Acceptance:

- A sentinel's next turn does not start until its previous turn is committed at the hub.
- When the runner running a sentinel goes silent, the turn it held is granted again, and the agent carries on.

### Woken, then at rest (product:wake-react-stop)

An agent is not left running. An event it is declared for wakes it, such as a chunk escalating or a standing
`needs-human`, or so does a regular interval, such as checking that the factory is not stalled. It works through what is
queued for it, then stops. Events arriving while it sleeps or works are queued for it, not lost and not run in
parallel.

Acceptance:

- A chunk escalating wakes the escalation agent, which reacts and then stops.
- Two escalations arriving together are handled one after the other by the same agent.
- An agent declared on an interval wakes on that interval with nothing else prompting it.

### One continuing conversation (product:resumable-conversation)

Each wake resumes the agent's conversation rather than starting cold, so it remembers what it already looked into. The
conversation is the turns the factory holds, so any volunteer can resume it, not only the runner that last ran it. When
that conversation is compacted or cleared is the factory's decision, not the session's
(`decision:sentinel-agents/conversation-home`).

Acceptance:

- An agent woken a second time knows what it did on the first wake.
- An agent woken on a different runner from its last wake knows what it did there.
- The factory compacts or clears an agent's conversation according to its own policy.

### Runners volunteer (product:runner-volunteers)

A runner declares that it will run sentinel agents, and the hub hands their sessions only to runners that have
volunteered.

Acceptance:

- A runner that has not volunteered never runs a sentinel agent.
- With no volunteer available, the agent's queued wakes wait and the operator can see that they are waiting.

## Open questions

- **Product owner:** What an agent may do on its own authority versus propose for a human, starting with the escalation
  agent: whether it may requeue, stop, or re-route a chunk, or only diagnose. `epic:agent-identity` would answer it
  with an altitude per agent.
- **Product owner:** Whether blizzard ships the escalation agent and the stall check as built-in roles or as examples a
  tenant copies.
