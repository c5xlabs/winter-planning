# Agent identity and capabilities decisions

## The altitudes (decision:agent-identity/altitudes)

**Open.** Which altitudes exist and what each reaches. Project and tenant are the first two; the tenancy model has roles
per tenant but none per project, so what a person must hold to queue project-altitude work is unsettled, as is whether
hub altitude exists for an agent at all. **Decider:** product owner.

## When the queuer's authority changes (decision:agent-identity/queuer-leaves)

**Open.** What happens to queued or running work when the person who queued it is demoted, loses their membership, or
leaves: whether the work keeps the authority it was queued with, is narrowed to the person's new role, or stops.
**Decider:** product owner.
