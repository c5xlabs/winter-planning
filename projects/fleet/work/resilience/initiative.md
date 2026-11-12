---
initiative: resilience
refinement: outline
epics:
  - provider-outages
---

# Resilience

A fleet that keeps its night when the world around it misbehaves.

## The problem

The fleet runs unattended on things it does not own: model providers, coding harnesses, forges, the machines its runners
sit on. When one of them falters, the fleet has no concept for a condition that is environmental rather than a piece of
work gone wrong, so it reads the trouble as failure, escalates, and spends the operator's morning on cleanup.

## The outcome

Trouble outside the fleet is recognized as trouble outside the fleet: ridden out, routed around where another route
exists, and surfaced once, plainly, when it outlasts the fleet's patience. `epic:resilience/provider-outages` makes that
move for the likeliest case, a model provider that stops answering.
