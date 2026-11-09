---
epic: lab/specimen
refinement: outline
status: planned
---

# Specimen

Part of `initiative:lab`; its requirements are inline below.

## The problem

A seed written by hand answers only the questions its author thought to ask, and an experiment run against one piece of
work says nothing about the next. `persona:harness-engineer` needs real work to run trials against, in quantity, without
any trial being able to see how that work actually turned out.

## The outcome

Seeds are drawn from real work. A **specimen** is a recorded chunk captured at the moment it entered the bench: the
artifacts it held, the commit its worktrees stood on, and the work items it wraps. The engineer collects specimens from
the fleet's own history (the chunks that delivered, the chunks that escalated, the ones that looped five times through
review) and runs every arm of an experiment against every specimen. The trials pair naturally: each specimen meets each
arm, so the hard specimens are hard for every arm alike and their difficulty cancels out of the comparison.

The fleet's own history is what makes this compound. Every chunk that delivers is a specimen with a known good outcome
attached, the change that actually landed, and the collection of them becomes a benchmark of the project's own work that
grows every night.

## Requirements

### Specimens from the fleet's history (product:recorded-specimens)

An engineer captures a specimen from any recorded chunk at the moment it entered a bench, with the artifacts it held,
the commit its worktrees stood on, and the work items it wraps, and a delivered chunk's specimen carries the change that
actually landed.

Acceptance:

- An engineer captures a specimen from a delivered, an escalated, or a looping chunk.
- A specimen of a delivered chunk shows the change that landed.

### The past stays sealed (product:sealed-specimens)

A worker re-planning a chunk that already delivered cannot read the answer: the worktree is cut at the specimen's
commit, with nothing later reachable from it, and no trial sees another trial's branches, findings, or artifacts. A
trial never wraps its work item the way a live chunk does, so a specimen of work still in flight can be studied without
disturbing it.

Acceptance:

- No commit later than the specimen's is reachable from a trial's worktree.
- No trial can read another trial's branches, findings, or artifacts.
- A trial on a specimen of in-flight work leaves the live chunk untouched.

### Recorded answers replayed (product:replayed-answers)

Where a trial asks a human a question, the specimen's recorded answer is replayed to every arm alike, so a person's mood
on a given evening never decides an experiment.

Acceptance:

- Every arm asking the same question of a specimen receives the same recorded answer.
