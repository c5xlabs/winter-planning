---
name: traced-selection
depends-on:
  - traced-adapter-resolution
code:
  - blizzard/src/blizzard/runner/lifecycle/session.py::HarnessSelector.select
  - blizzard/src/blizzard/runner/lifecycle/session.py::SessionResolver.session_stamps
---

# Traced selection

Extracts the per-member check from `HarnessSelector.select` and adds a selection trace giving every member one outcome
([resolution.md](../spec/resolution.md) §Harness selection). Delivers the spawn-path agreement of
`engineering:traced-resolution` and the candidate lines of `product:session-pools`.

## Change

- `HarnessSelector.trace` gives each set member `selected`, `passed-over`, or `skipped` with the selector's reason, and
  the basis `acceptable-set` or `runner-default`.
- `select` and `SessionResolver.session_stamps` are built on the traced forms and keep their signatures.

## Acceptance

- For every configuration fixture and authored session in the suite, the traced value equals what
  `SessionResolver.session_stamps` returns for a fresh mint, over fixtures that exercise every source in the provenance
  vocabulary.
- An empty set yields the runner default as the only candidate; a set with every member skipped yields no selection.
