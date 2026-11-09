# Lab runner requirements

The requirements of `epic:lab/lab-runner`.

## Runners say which workspace they offer (product:workspace-kind)

A runner declares whether it offers a project workspace or a lab workspace. Only a runner offering a lab workspace
claims trials, and a project runner never claims one.

Acceptance:

- A trial is never claimed by a runner offering a project workspace.
- A chunk working on a project is never claimed by a lab runner.

## Arm workspaces materialized exactly (product:arm-workspaces)

An experiment names each arm's workspace definition by a reference the hub stores and never interprets; the lab runner
resolves it, materializes it on demand, and reports a fingerprint of what it primed each trial from, so the hub records
which definition every trial ran in without ever reaching into the runner. How a runner learns an arm's definition is
`decision:lab/lab-runner/arm-definition`.

Acceptance:

- Every trial records the fingerprint of the workspace it ran in.
- Two arms meant to be identical record the same fingerprint.
- A trial whose arm changed while it ran is set aside.

## Clean room, certified (product:clean-room)

Before an experiment's first wave, the lab runner certifies that a trial session loads the arm's instructions, skills,
and agents and nothing else: not the lab's own, not a stray file in a directory above it, and not a user-level setting
that belongs to no arm. User-level harness configuration is pinned rather than erased, since the fleet's own workers run
with one, and which configuration a lab runner pins is part of the definition an arm can vary.

Acceptance:

- An experiment whose trial session loads an instruction file from outside its arm does not start, and the lab says
  which file.
- The user-level configuration a trial ran with is recorded with its arm's definition.

## Open questions

- Whether two workspaces can be compared across two hosts. Arms on different machines differ in their load, their
  subscriptions, and their user-level configuration as well as their workspace. One lab workspace holding every arm is
  the clean comparison; a comparison across hosts should be reported as the bundle it is.
