---
epic: running-environments/provider-docker
refinement: outline
status: planned
---

# Run native first-party container services

Part of `initiative:running-environments`; product requirements live in [requirements.md](./requirements.md). Supports
engineers who use Docker Compose for isolated environment services and persistent shared infrastructure.

## The problem

Engineers need container isolation and real health checks without Python helpers or teardown scripts that interpret a
different declaration from startup. A common log command must also follow every selected container service.

## The outcome

The engineer can manage Docker-backed scopes through native first-party behavior with precise targeting, trustworthy
health, and explicit volume retention. Delivery is independent of the tmux and process-compose provider epics.

## Depends on

- `epic:running-environments/service-orchestration`

## What this epic is not

- Docker, Compose, image construction, or a rewrite of the user's compose files and application programs.
- Delivery of the tmux or process-compose provider.
