---
name: hub-sessions-routes
depends-on:
  - traced-hub-merge
code:
  - blizzard/src/blizzard/hub/api/fleet.py::router
  - blizzard/src/blizzard/wire/graph.py::GraphSessionView
---

# Hub sessions routes

Adds `GET /api/fleet/sessions` and `GET /api/fleet/chunks/{chunk_id}/sessions` and their wire model
([hub-sessions.md](../spec/hub-sessions.md)). Delivers `engineering:hub-merge-read` and serves `product:session-pools`.
Waits on the open question of how a `404` for an unknown name differs from a missing route.

## Change

- The fleet router answers both routes with `list[MergedSessionView]`, built through `EffectiveSession.merge`.
- `/sessions` reports every effective graph, or the named one, with empty chunk defaults.
- `/chunks/{chunk_id}/sessions` merges the pinned graph with the chunk's defaults and adds the bare-lineage entry.
- `openapi/hub.openapi.json` gains both routes. The change is additive.

## Acceptance

- A runner token without a lease is answered by both routes.
- A request without a runner token is refused.
- An unknown graph or chunk returns `404` naming what did not resolve.
- Sessions come back graphs by name, in authored order, with each field's `graph`, `chunk`, `value`, and `source`.
