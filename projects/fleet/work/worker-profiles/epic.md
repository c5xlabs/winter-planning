---
epic: worker-profiles
refinement: outline
status: planned
---

# Worker profiles

A standalone epic; its requirements are inline below.

## The problem

A graph binds session settings per node, which is right: a reviewer and a builder should not run the same way. What is
wrong is that every node spells its settings out, so changing how every reviewer in the fleet runs is one edit per node,
and a graph author comparing two configurations is comparing two piles of restated numbers.

## The outcome

A profile is the indirection that fixes it: settings named once and inherited whole.

## What this epic is not

It refuses the identity layer that tends to grow around profiles (roles, reporting lines, an org chart of agents), which
answers no question blizzard's fact log cannot already answer.

## Requirements

### A profile as a named bundle (product:named-profile)

The model tier, effort, context ceiling, rotation thresholds, and permission posture a session runs under are carried
under one name (`decision:worker-profiles/profile-home`).

Acceptance:

- Changing a profile changes how every node naming it runs, with one edit.

### A node spells it out or names one (product:node-names-profile)

Nodes keep the freedom to state their own settings; naming a profile inherits them whole, and the two forms are equally
legitimate (`decision:worker-profiles/field-override`).

Acceptance:

- A graph author can give a node its own settings or a profile name, and both run.

### The name on the record (product:profile-on-fact)

A run records the profile it ran under, so a runner and a fact have a name to hold and an operator can ask what every
reviewer-profile session cost last week.

Acceptance:

- The operator finds what every session under a given profile cost over a week.

## Open questions

- **Product owner:** What a changed profile does to comparison across nights, since two runs under the same profile name
  may no longer have run the same way.
