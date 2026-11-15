---
name: scenario-context
depends-on:
  - proof-state-changing
---

# Contributor document for recorded scenarios, and its matrix rows

Adds the scenario document, the `workspace:scenario` row, and `tool:record` to the code repository's context tree, as
[contributor-context](../../../parallel-workspaces/component-runtime/spec/contributor-context.md) reserves them. It
restates [scenario-format.md](../spec/scenario-format.md) and the Flows of
[record-and-replay.md](../spec/record-and-replay.md) for a contributor, under
`engineering:parallel-workspaces/component-runtime/workspace-development-context`.

## Change

- `context/verification/scenarios.md` exists: the directory layout of one scenario, how to add and record one, how to
  read a replay failure by dimension, how to keep the tree, how to register a deviation, how to mark a facility, and the
  rule that only `cargo dev record` starts the reference tool and only inside a temporary fixture tree.
- `context/verification/index.md` and the routing table of `AGENTS.md` link it; `matrix.md` gains the
  `workspace:scenario` row in its Commands part and `tool:record` in its Tools part, as the contributor-context spec's
  Verifiability matrix gives them.
- Files touched: `context/verification/{scenarios,index,matrix}.md`, `AGENTS.md`.

## Acceptance

- The architecture test passes: the document is reachable from `AGENTS.md` and its links resolve.
- The Commands part of the matrix has exactly seven ids, `workspace:scenario` among them with the command
  `cargo scenarios`, which exits 0 on the checkout; the Tools part has `tool:format`, `tool:keep-tree`, and
  `tool:record`.
- Every command line in the document's fenced blocks is `cargo scenarios`, `cargo dev record`, or `cargo dev gate` with
  arguments the record and replay spec defines.
- The proof scenario ids the document uses as examples exist under `scenarios/`, and its example of a registered
  deviation is `repo-remove/removes-middle-block`.
- `cargo dev gate full` exits 0.

## Out of scope

- A catalogue of scenarios per command: each command's owning epic adds its scenarios.
