---
epic: config/model-resolution
refinement: outline
status: planned
---

# Model resolution

Part of `initiative:config`; its product requirements are in [requirements.md](./requirements.md).

## The problem

A graph author writes a capability tier and an effort, never a model, and that is what lets one graph run under either
harness. The price is that the model a session actually runs on is decided at a distance, in several places that each
have a say: the graph's session pool pins a tier and an effort; the chunk may carry defaults of its own; the runner's
configuration maps each tier to a concrete model per harness and renames efforts on the way through; and each adapter
brings built-in defaults underneath all of it, which quietly answer for any tier the runner leaves unmapped.

`persona:harness-engineer` is the person this lands on. Their craft is moving one of those settings and reading what
changed, and today the only way to know what a pool will run on is to hold the graph, the runner file, and the adapter
defaults open side by side and do the resolution in their head. It is the same reconstruction they would be doing the
morning after a bad night, only in advance.

## The outcome

The runner answers that question itself. One command, on the machine the runner lives on, prints the resolution the way
the runner will perform it: every layer that holds a value, which value won, and why. It is the same answer
`initiative:config` promises across the whole configuration surface, a lookup rather than an investigation, given early
to the one corner of it where the answer changes what a night costs.

## What this epic is not

The command reports; it changes nothing. It reaches no web surface and no board view: a place to look at this in the
browser is a later decision, and a read-only CLI is the right first home for a question asked mostly by the person
editing the files it describes.

It stops at the session a worker is spawned into. The models of the subagents that session goes on to spawn are set by
the workspace the worker runs in, not by blizzard, and blizzard only observes them in the transcript; that half of the
picture is winter's to report. Joining the two into one view, blizzard and winter configured and inspected together, is
a larger question about how the two products meet, and it is not asked here.
