---
name: standalone-refs
depends-on:
  - epic:parallel-workspaces/repository-catalog
code:
  - winter-workspace/crates/core/src/config.rs::load
---

# The declared `ref` of each loaded standalone

Adds `ref` to the `StandaloneEntry` the catalog loads, as the Domain of [standalone-pins.md](../spec/standalone-pins.md)
gives it, so the pin flows can select pinned standalones in merged order with their catalog location. The load's value
rule for `ref` is already the delivered configuration model's.

## Change

- `StandaloneEntry` gains `ref: Option<String>`: the entry's string `ref`, `None` when absent. The load's refusals,
  including a non-string `ref`, do not change.
- Nothing reads it yet.

## Acceptance

- Unit tests of the load: a standalone with `ref = "v1"` carries it; one without carries none; a local-file standalone
  with a `ref` carries it after the shared file's entries.
- `cargo dev gate slice` exits 0.
