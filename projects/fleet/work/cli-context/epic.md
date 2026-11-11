---
epic: cli-context
refinement: outline
status: planned
---

# CLI context

A standalone epic; product requirements live in [requirements.md](./requirements.md).

## The problem

A person asks an agent in a participating workspace why a change has stayed in delivery. The workspace does not explain
its relationship to the factory, and the installed CLI offers no route into the operating guidance. The agent searches
for a development checkout, confuses a local test hub with the real fleet, or acts without knowing which machine and
role can perform the operation. Operating Blizzard should not require knowing where Blizzard itself is developed.

## The outcome

The installed Blizzard CLI answers "How do I use Blizzard?", one requested topic at a time. It teaches people and agents
how the product works and how to use its commands; it does not return the factory's artifacts, configured prompts, or
live work data. A workspace supplies its own connection and participation facts and a short instruction leading into
that guidance. An interactive agent can inspect the right fleet without assuming it is a leased worker or that it may
administer the runner host. The guidance ships with the CLI, works offline, and is useful outside Winter workspaces.

## What this epic is not

- A separate context repository or an installation of Blizzard's developer conventions into customer workspaces.
- A router over arbitrary local files, project documents, or installed extensions.
- A saved-connection feature, authentication redesign, or source of live fleet state.
- A replacement for artifact reads, graph declarations, workspace-prompt management, or runtime worker assignments.
- A rewrite of Blizzard's Python CLI or services.
- Automatic edits to participating workspaces, deployment changes, or a new documentation website; the website is owned
  by `epic:documentation`.
