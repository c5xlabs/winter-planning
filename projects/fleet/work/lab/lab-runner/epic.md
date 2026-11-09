---
epic: lab/lab-runner
refinement: outline
status: planned
---

# Lab runner

Part of `initiative:lab`; its requirements are in `requirements.md`.

## The problem

A runner brings its own workspace, and the hub never dictates which one or at what commit. That is deliberate and it
stays true: a project workspace is long-lived and shared, many chunks pass through it one after another, and a fleet
that cut a fresh, pinned workspace for every chunk would spend more on isolation than on work. Ten runners with one
environment each would satisfy a purist and waste a machine. But in the lab the workspace is often the thing being
measured, so a trial run in whatever workspace a runner happens to hold measures nothing `persona:harness-engineer` can
trust.

## The outcome

The lab is the one place where a workspace is pinned. A lab runner brings a **lab workspace**: a meta-workspace that
holds many workspaces, one per arm, each cut from a versioned definition (the workspace's commit and the exact commit of
every extension it installs) and initialized in place with its own environments for trials. It is the same move winter
already makes one level down: where a project workspace is one workspace with many feature environments, a lab workspace
is one lab with many workspaces. The lab workspace is itself versioned like any workspace, and the workspaces inside it
can be any winter workspace whose context is complete in its own repository.

A runner therefore offers one of two things: a project workspace, for chunks working on a project, or a lab workspace,
for trials. Project runners never claim trials, so an experiment can never pin, disturb, or borrow the environment real
work runs in, and a trial only ever runs where its arm's workspace can be materialized exactly. The lab runner is the
manual winter lab with the fleet's claiming, leasing, and fact log around it, which is why the manual lab comes first.
