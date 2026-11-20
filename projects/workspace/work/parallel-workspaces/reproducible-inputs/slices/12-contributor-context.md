---
name: contributor-context
depends-on:
  - freeze
  - fingerprint-command
---

# Contributor guidance for commands that change checkouts

Brings the repository's own guidance up to the commands this epic adds: `ws update` fetches, resets branches, stashes,
and writes the configuration and the lock of the workspace it finds, beyond what `repo remove` alone did.

## Change

- The README's paragraph on where the binary may run, and the matching rationale in `AGENTS.md`, name `ws update` and
  what it changes beside `repo remove`.
- The Modules table of `structure.md` lists the pin and content-identity modules and what each may call.

## Acceptance

- `cargo test -p winter-workspace-dev` exits 0, covering the context-tree rules.
- `cargo dev gate slice` exits 0.
