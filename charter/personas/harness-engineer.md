# The harness engineer (persona:harness-engineer)

The suite's mechanic, tuning the harness and the system around it and reading outcomes across many runs.

## Who they are

The suite's mechanic. They care less about any single task's output than about the *ecosystem of outputs*: the
distribution of outcomes across many runs, and how that distribution shifts when a parameter moves. When the automated
system goes wrong, their first instinct is not to fix the one incident but to reconstruct exactly what happened and ask
what setting would have prevented the class.

## What their work looks like

Tuning, mostly. They choose the coding harness and the skills, agents, and context it receives, they write extensions in
the language they prefer, and they adjust the settings that govern how agents run. They change one thing, run the
agents, and read the results. Their enemy is anything that makes two runs incomparable or an incident unreconstructable.

## What they need from the suite

Every operational constant lives in configuration, changeable without touching code, because a constant buried in code
is a dial they cannot turn. Every command an agent runs gives a result it can parse or a reason it can act on, so a
failure is a fact they can study rather than a silence. The harness is theirs to choose, and an extension they write
keeps working across releases. A parameter change is judged against real outcome data, never against vibes.

## What serves them

The platform's versioned extension contract and its one command shape; Workspace's support for any harness with the same
skills, agents, and context; and Fleet's facts-only record of what happened. What they need from Fleet in particular is
set out in [the Fleet vision](../../projects/fleet/charter/vision.md). This persona is the reason operational constants
must land in configuration, not in code.
