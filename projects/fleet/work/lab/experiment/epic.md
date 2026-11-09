---
epic: lab/experiment
refinement: outline
status: planned
---

# Experiment

Part of `initiative:lab`; its requirements are in `requirements.md`.

## The problem

A bench gives `persona:harness-engineer` one trial at a time, started by hand. One trial is an anecdote: the engineer
needs every arm run against the same work, enough times to separate a real difference from noise, without spending more
than the question is worth or crowding out the fleet's real work.

## The outcome

A lab graph describes one traversal of a bench, and it never describes more. Running a bench twenty times is not
something the graph does; it is something the experiment asks for. A graph that looped twenty times would run its trials
one after another on one runner, pile twenty attempts into the same artifact series where no reader could tell them
apart, and turn the count of trials into a property of an immutable definition, so that asking for five more would mean
minting a new graph. The graph stays the shape of a single trial, and how many trials, of what, and in what order belong
to the **experiment**, a record of its own on the hub and a sibling to the routine rather than a kind of chunk or work
item.

Each **trial** is an ordinary chunk, and that is the most important choice in the initiative. A chunk already has
everything a trial needs: a runner claims it under a lease, it is fenced against a stale attempt, it records every
movement and every token as facts, its transcripts are shipped home, and a crashed runner hands it to another. Nothing
in that machinery has to learn about experiments.

The experiment is the conductor the graph model does not have. No chunk can wait on many, so the experiment does the
waiting: it mints trials in waves, mints the comparisons that need two finished trials, reads what it has between waves,
and stops on a clear answer, a clear non-answer, or an empty budget. Twenty runs is a ceiling the engineer sets, not a
quota the lab must fill. It is what turns the bench's hand-started trials into a measurement.
