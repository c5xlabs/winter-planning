---
epic: agent-identity
refinement: outline
status: planned
---

# Agent identity and capabilities

A standalone epic; its requirements are inline below.

## The problem

Every agent the fleet runs has the same identity: the runner's. A runner may do runner things, claim a chunk, report on
it, hand back artifacts, and nothing else; it is not signed in to the hub as anyone, and it should not be. That
confinement is right for a builder. It leaves the work that is about the factory, rather than inside it, with no way to
be done by an agent. Gardening already works around this well: a worker submits its findings as artifacts, and a landing
node turns them into proposals. But tending the garden, watching the proposal docket, adding a scope, setting up a
repository or a cadence, or standing up a new project all need an agent that acts on the hub, and today only a person
can.

## The outcome

Work can be queued to run as a different kind of agent: not a builder, but an agent-operator, carrying a hub credential
of its own at a declared altitude, such as project or tenant. An agent at project altitude may add a scope to its
project; one at tenant altitude may add a project. The person who queues the work bounds it: an admin can queue work
that acts as an admin, and a contributor cannot. With that, the factory can set up a new project on its own, and begin
to build its own products.

## What this epic is not

- A loosening of the builder. A worker's confinement stays exactly as it is, and an agent-operator's credential never
  reaches a builder's session.
- A way across the tenant wall. No agent acts in a tenant other than the one its work was queued in.

## Requirements

### An identity per kind of work (product:work-identity)

Work declares the kind of agent it runs as, a builder or an agent-operator, and the agent runs with that identity and no
other.

Acceptance:

- Work declared to run as an agent-operator acts on the hub through its own credential.
- Work declared to run as a builder cannot act on the hub, exactly as today.

### Altitudes (product:altitudes)

An agent-operator's reach is a declared altitude, each a ceiling on what it may change: a project altitude reaches only
its project, and a tenant altitude reaches the tenant's projects and settings (`decision:agent-identity/altitudes`).

Acceptance:

- A project-altitude agent adds a scope to its project and is refused when it reaches for anything in its tenant beyond
  that project.
- A tenant-altitude agent adds a project to its tenant.

### Never above the person who queued it (product:bounded-by-queuer)

An agent-operator's authority is borrowed from the person who queued its work and can never exceed theirs. A person may
queue work at their own altitude or below it, never above (`decision:agent-identity/queuer-leaves`).

Acceptance:

- A tenant admin queues tenant-altitude work, and it runs.
- A contributor's attempt to queue tenant-altitude work is refused.

### The credential lives with the work (product:scoped-credential)

The credential an agent-operator uses is issued for its work, held only for as long as that work runs, and gone when it
ends, so an agent's authority is never a standing key someone can pick up later.

Acceptance:

- When the work ends, its credential no longer acts on the hub.

### Every act on the record (product:attributed-acts)

What an agent-operator changes is recorded as the agent's act, on behalf of the person who queued its work, so the
operator can always answer who changed a setting and why.

Acceptance:

- A change an agent made shows the agent, the work it was doing, and the person who queued that work.

## Open questions

- **Product owner:** Whether any agent may stand at hub altitude, creating tenants as a hub administrator can, or
  whether that altitude stays reserved to people.
- **Product owner:** Which acts an altitude grants: whether a tenant-altitude agent may manage memberships and roles, or
  only the tenant's configuration.
- **Product owner:** How `epic:sentinel-agents` takes its authority from this epic. A sentinel agent declared per
  project would run at project altitude and never reach the tenant, but its wakes are queued by events, not people, so
  whose authority bounds it: perhaps the admin who defined it.
