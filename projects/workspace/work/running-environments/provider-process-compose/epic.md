---
epic: running-environments/provider-process-compose
refinement: outline
status: planned
---

# Run native first-party supervised processes

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
engineers who want headless local processes supervised by process-compose through the common service interface.

## The problem

Engineers need repeatable local process supervision without Python orchestration, shared sockets, or orphan cleanup that
guesses ownership after a daemon or environment disappears.

## The outcome

The engineer can operate one isolated process-compose instance per scope, preserve unchanged processes on repeated up,
and recover stale instances with bounded, ownership-safe behavior. This provider is deliverable independently of tmux
and Docker support.

## Depends on

- `epic:running-environments/service-orchestration`

## What this epic is not

- A replacement process supervisor or a rewrite of user service commands.
- Delivery of tmux or Docker support, or an unsupported promise that Linux recovery works unchanged on every OS.
