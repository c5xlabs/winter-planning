---
epic: running-environments/provider-tmux
refinement: outline
status: planned
---

# Run native first-party services in tmux

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
engineers who need visible terminal sessions and interactive services through the common service interface.

## The problem

Engineers using tmux need its terminal behavior without Python launch helpers, leaked scope values, or recovery that
kills another workspace's same-named environment.

## The outcome

The engineer can launch, inspect, restart, and stop tmux-backed services through a native first-party provider, retain
interactive access and useful logs, and recover only resources belonging to their scope. This provider can be delivered
and used without either other first-party provider.

## Depends on

- `epic:running-environments/service-orchestration`

## What this epic is not

- A tmux replacement or a rewrite of the user's layout hooks, shell profiles, or service commands.
- Delivery of the Docker or process-compose provider.
