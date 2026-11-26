---
name: env-name-rule
depends-on:
  - module-scaffold
---

# The environment name rule

Adds `env_name::check` and `Reserved::of` of [env-names.md](../spec/env-names.md), The rule and Reserved names: the pure
test every name that addresses a feature environment passes before a command reads Git or writes anything. Builds the
refusal half of `product:environment-targets` and the one-component guarantee of
`engineering:worktree-effect-confinement`, under `decision:parallel-workspaces/environment-lifecycle/env-name-grammar`,
a registered deviation. The commands apply it in `ws-init-env`, `ws-init-all`, and `ws-destroy-command`.

## Change

- `crates/core/src/env_name.rs` holds `check` and `Reserved`: the five tests in the spec's order, the first failing one
  the error, with the branch-name test written out so no git process runs.
- `fold` is the Unicode full case folding of the NFC normalisation, applied to test 5 only; `crates/core/Cargo.toml`
  gains the crates that provide both.
- `Reserved::of` takes the first component of each loaded standalone's path, else of its resolved name, and always
  reserves `workspace` and `projects`.
- No command calls it yet.

## Acceptance

- Unit tests: `""`, `a/b`, `.`, `..`, `.hidden`, `.winter`, `workspace`, and `projects` are each refused with their
  variant before any later test runs; `Projects`, `WORKSPACE`, and the NFD spelling of a standalone's first component
  are refused as `Reserved`; a standalone at `ext/tools` reserves `ext`.
- Unit tests: each `BranchFault` is answered for one name that holds it, and a name holding two faults answers the first
  in the spec's order.
- Unit tests: a name of non-ASCII letters other than the forbidden characters passes; `feature-1` and `alpha` pass.
- A component test with real git: over a fixed list of one-component names covering every fault, `check`'s branch-name
  answer agrees with whether `git check-ref-format --branch <name>` exits 0.
- `cargo dev gate slice` exits 0.
