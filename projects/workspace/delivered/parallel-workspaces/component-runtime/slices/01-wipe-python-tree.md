---
name: wipe-python-tree
---

# Wipe the reference tool from the branch

Removes every tracked file of the reference tool and rewrites the root documents and tool configuration, delivering
`engineering:rust-only-branch` as laid out in [repository-layout.md](../spec/repository-layout.md), Root after the wipe.
`decision:parallel-workspaces/component-runtime/retained-root-documents` keeps `LICENSE` and `PRINCIPLES.md`;
`decision:parallel-workspaces/component-runtime/ci-workflow` removes the workflow.

## Change

- `tools/`, `context/`, `skills/`, and `.github/workflows/ci.yml` are deleted; `.github/` holds nothing afterwards.
- `README.md`, `AGENTS.md`, `CLAUDE.md`, `.gitignore`, `dprint.json`, and `.rumdl.toml` hold the content the spec's
  table gives each. `AGENTS.md` is its title and the one-sentence description: no standing rule of
  [contributor-context.md](../spec/contributor-context.md), Entry point, is true of this tree yet, and it has no routing
  table.
- `LICENSE` and `PRINCIPLES.md` are untouched.
- Files touched: the trees and files named above, and nothing else. No Rust or Cargo file exists after this change.
- The tree has no `Cargo.toml`, so the change is admitted on the acceptance checks below alone, per
  [gates.md](../spec/gates.md), Which check admits a change.

## Acceptance

- `git ls-files` lists exactly `.gitignore`, `.rumdl.toml`, `AGENTS.md`, `CLAUDE.md`, `LICENSE`, `PRINCIPLES.md`,
  `README.md`, and `dprint.json`.
- `git diff --stat` against the parent shows no change to `LICENSE` or `PRINCIPLES.md`.
- None of the six rewritten files contains a scope token of [contributor-context.md](../spec/contributor-context.md),
  System, matched as that section says.
- `dprint.json` parses as JSON and names the three plugin versions of the spec's table; `.rumdl.toml` parses as TOML.

## Out of scope

- Any Cargo file: `cargo-workspace`.
- The standing rules, the context tree, and the routing table of `AGENTS.md`: the slices that make each rule true, and
  `contributor-context`.
- A CI workflow: `gates`.
