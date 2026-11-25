---
name: agents-md
external: github:paul-gross/winter#250
depends-on:
  - managed-blocks
  - module-scaffold
  - epic:agent-ready-operation/context-navigation
---

# The eager safety block of AGENTS.md and the CLAUDE.md shim

Adds `place`, `refresh`, and `shim` of [managed-content.md](../spec/managed-content.md), Interfaces, The eager safety
block and the shim, placing `eager_text()` of `epic:agent-ready-operation/context-navigation`. Delivers the
`AGENTS.md`/`CLAUDE.md` bullets of `product:managed-content-ownership` and the placement bullet of
`product:create-unforked-workspace`, under `decision:parallel-workspaces/workspace-bootstrap/creation-ownership`.

## Change

- `crates/core/src/agents_md.rs` holds the three entries over the file port; each writes through one `update`, and a
  conflict writes nothing.
- They are re-exported from `lib.rs`; no command calls them yet.

## Acceptance

- Component tests: `place` in an empty directory writes the block alone; on an `AGENTS.md` without the block it prepends
  the block and one blank line and keeps every other byte; on one with a current block it writes nothing; on one with an
  outdated block it replaces the block alone; on a malformed block or non-UTF-8 content it writes nothing and reports
  the conflict.
- Component tests: `refresh` is `Absent` with no `AGENTS.md` and with one that holds no begin marker, non-UTF-8 content
  included, and otherwise behaves as `place`.
- Component tests: `shim` writes `@AGENTS.md\n` when `CLAUDE.md` is absent, leaves a file holding that line as
  `Present`, and leaves any other file untouched as `Conflict`.
- A component test: the block's body is `eager_text()` line for line.
- `cargo dev gate slice` exits 0.
