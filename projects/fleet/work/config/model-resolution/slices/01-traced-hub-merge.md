---
name: traced-hub-merge
code:
  - blizzard/src/blizzard/hub/domain/execution/envelope.py::EffectiveSession
---

# Traced hub merge

Gives the hub merge a declaration-level form that records each field's source ([resolution.md](../spec/resolution.md)
§The hub merge). Delivers the merge half of `engineering:traced-resolution` and the `graph`, `chunk`, and `unset` terms
of `product:cell-provenance`.

## Change

- `EffectiveSession.merge(declaration, defaults)` merges a declaration, or none, with chunk defaults and records a
  source for `model`, `effort`, and `harnesses`.
- `EffectiveSession.of` delegates to it and keeps its signature and result.
- The node envelope keeps its shape.

## Acceptance

- One test per field shows the source change from `graph` to `chunk` exactly when the declaration's value is empty or
  null, beside the existing test pinning the precedence.
- A bare lineage takes all three fields from the chunk; empty chunk defaults give `unset`.
