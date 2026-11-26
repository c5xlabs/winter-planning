---
name: env-selection
depends-on:
  - module-scaffold
---

# The environment reaching rules

Adds `env_selection::reach`, `known_env`, and `EnvRule` of [env-selection.md](../spec/env-selection.md): the `Present`
and `Named` rules that list the environments a command's patterns reach, and the unknown-literal refusal. This is the
contract `ws destroy` selects through here and the `<env>/<repo>` matcher of `epic:parallel-workspaces/coordinated-git`
builds on. Delivers the environment half of `product:target-patterns` and the glob and literal bullets of
`product:environment-targets`, under `decision:parallel-workspaces/environment-lifecycle/discovery-authority` and
`decision:parallel-workspaces/target-selection`, whose unknown-literal refusal is a registered deviation.

## Change

- `crates/core/src/env_selection.rs` holds `EnvRule`, `known_env`, `reach`, and `ReachError`; `reach` drops duplicate
  halves, refuses the first literal half that names no environment, then lists by the rule.
- The environments present are `environments::present` as built; globs are `pattern_match::is_glob` and `fnmatchcase` as
  built.
- No command calls it yet.

## Acceptance

- Unit tests over `MemoryFiles`: `known_env` holds for a name the registry records with no directory, for a directory
  the registry does not record, and for a link to a directory; it fails for a regular file and for nothing at the path.
- Unit tests, `Named`: literal halves come first, sorted by code point, each reached whether or not it is present; then
  the present environments a glob matches that are not literals, sorted; a repeated half reaches once.
- A unit test, `Named`: with no glob half, a candidate directory that cannot be listed is never read and `reach`
  succeeds; with a glob half, the same workspace ends `ReachError::Internal`.
- Unit tests, `Present`: environments come in `present`'s index order; a literal that passes `known_env` and is not
  present reaches nothing.
- Unit tests, both rules: of two unknown literals, the first in the order given is the `UnknownEnv` error, and nothing
  is listed.
- `cargo dev gate slice` exits 0.
