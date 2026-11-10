---
name: operator-docs
depends-on:
  - daemon-report
---

# Operator docs

Documents `blizzard runner models` for operators and agents ([command.md](../spec/command.md)).

## Change

- `docs/deployment/control-verbs.md` describes the verb, its options, and its exit codes.
- `docs/deployment/worker-spawn.md` points from the model and effort alias tables to the verb.

## Acceptance

- The docs build and link checks pass.
