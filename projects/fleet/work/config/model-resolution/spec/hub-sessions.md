# Hub sessions read

## Interfaces

Two routes on the fleet router (`prefix="/api/fleet"`), behind its `require_runner_principal` dependency. A runner token
is enough; no lease is needed.

```text
# engineering:hub-merge-read
GET /api/fleet/sessions[?graph=<name>]
  200  list[MergedSessionView]
       # Every effective graph (per name, the newest graph that is not retired), or only the named one.
       # Graphs by name, sessions in authored order. Chunk defaults are empty: every source is graph or unset.
  404  the graph name did not resolve; the body names it

GET /api/fleet/chunks/{chunk_id}/sessions
  200  list[MergedSessionView]
       # The chunk's pinned graph merged with this chunk's defaults (EffectiveSession.merge).
       # A runner-executed node on a bare lineage adds one entry with session = null.
  404  the chunk id did not resolve; the body names it

Both, without a runner token: refused by require_runner_principal.
```

`blizzard.wire.graph`, beside `GraphSessionView`:

```python
class SessionFieldView[T](WireModel):
    graph: T  # the declaration's value
    chunk: T  # the chunk's default; empty or null on /sessions
    value: T  # the effective value
    source: Literal["graph", "chunk", "unset"]


class MergedSessionFieldsView(WireModel):
    model: SessionFieldView[list[str]]
    effort: SessionFieldView[str | None]
    harnesses: SessionFieldView[list[str]]


class MergedSessionView(WireModel):
    graph_id: str
    graph_name: str
    session: str | None  # None: the chunk's bare-lineage entry
    fields: MergedSessionFieldsView
```

An older hub answers `404` on both routes. The runner reports that as `unsupported`, distinct from an unreachable hub
([command](./command.md#flows)).

## Open questions

- The wire model names are provisional; the fields are settled. Engineering answers.
- Both a missing route on an older hub and an unknown graph or chunk answer `404`. The body that lets the runner tell
  them apart (the fleet router's problem responses carry a type) is not settled. Engineering answers.
