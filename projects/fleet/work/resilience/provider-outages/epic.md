---
epic: resilience/provider-outages
refinement: outline
status: planned
---

# Riding out provider outages

Part of `initiative:resilience`; its requirements are inline below.

## The problem

At one in the morning a model provider stops answering. Nobody is awake, and the fleet, which has no concept for "the
weather turned bad", reads the silence as work that failed. Every chunk in flight escalates, the queue fills with
failures that describe nothing an operator can act on, and by breakfast the night is gone and the morning starts with
cleanup. It is the likeliest cause of the wedged fleet the charter's first success criterion forbids: an outage at one
in the morning currently costs the whole night.

## The outcome

The fleet does what the operator wanted: it waits, and then carries on. A night interrupted becomes a night delayed.

## Requirements

### An outage is not a failure (product:outage-not-failure)

A provider that is unreachable, or a rate limit that has closed the window, is recognized as an environmental condition
rather than as a chunk that went wrong, and nothing escalates on its account.

Acceptance:

- During a provider outage, no chunk escalates because of it.

### Waiting that keeps its place (product:hold-place)

The affected work keeps its position in the queue and resumes where it stopped once the provider answers again.

Acceptance:

- After the provider answers again, the affected chunks resume where they stopped, in their queue order.

### Waiting that says so (product:visible-waiting)

A fleet that is waiting looks different from a fleet that is idle. The operator who checks at six sees what the fleet is
waiting on and since when, without reading a log.

Acceptance:

- The operator sees what the fleet is waiting on and since when, without reading a log.

### Falling back before waiting (product:fallback-first)

Where another route exists the fleet takes it: down the model tiers first, and across coding harnesses once
`initiative:adapters` has given it a second one to fall back to.

Acceptance:

- When a lower model tier still answers, the work continues on it instead of waiting.

### A floor under the patience (product:bounded-wait)

Waiting is bounded (`decision:resilience/provider-outages/patience-bound`). An outage that outlasts the bound becomes one
ordinary escalation about the outage, not one per chunk.

Acceptance:

- An outage that outlasts the bound raises exactly one escalation, naming the outage.
