---
name: traced-adapter-resolution
code:
  - blizzard/src/blizzard/runner/harness/harness_shared.py::resolve_model
  - blizzard/src/blizzard/runner/harness/harness_shared.py::resolve_model_strict
  - blizzard/src/blizzard/runner/harness/claude_code/adapter.py::ClaudeCodeAdapter
  - blizzard/src/blizzard/runner/harness/opencode/adapter.py::OpenCodeAdapter
  - blizzard/src/blizzard/runner/harness/opencode_v2/adapter.py::OpenCodeTwoAdapter
---

# Traced adapter resolution

Adds traced model and effort resolution to the shared skeleton and every binding, and makes the existing `resolve_*`
methods projections of it ([resolution.md](../spec/resolution.md) §Adapter resolution). Delivers the model and effort
terms of `product:cell-provenance` and the per-harness rows behind `product:tier-table` and `product:effort-table`.

## Change

- `trace_model` walks the preference left to right and returns the value, its source, the winning entry, and each
  earlier entry tried.
- Each binding traces effort to `runner-alias`, `as-authored`, `dropped`, or `unset`, with the flag it becomes.
- `resolve_model`, `resolve_model_strict`, and `resolve_effort` return the traced `.value` with unchanged signatures.

## Acceptance

- For each binding, every row of the model and effort tables in the spec has a test giving its source.
- An unmapped tier on OpenCode traces `unmapped`; `xhigh` traces `dropped` on both harnesses.
- The existing resolution tests pass unchanged.
