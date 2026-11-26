---
name: target-patterns
depends-on:
  - module-scaffold
  - epic:parallel-workspaces/environment-lifecycle
---

# The `<env>/<repo>` selection

Adds `targets::parse`, `matches`, `select`, and `standalones` of [target-patterns.md](../spec/target-patterns.md): the
pattern grammar, the per-worktree match over the environments `reach` of
[env-selection.md](../../environment-lifecycle/spec/env-selection.md) lists, the name refusal of
[env-names.md](../../environment-lifecycle/spec/env-names.md), the missing-worktree and missing-standalone warnings, and
the `missing` entry and `target_missing` event of Missing targets in results. Both rules and the name check come from
`epic:parallel-workspaces/environment-lifecycle`. This is the selection every worktree command calls here and the
commands of `epic:parallel-workspaces/safe-history` call after this epic. Delivers `product:select-git-targets`'
selection bullets and `product:target-patterns` for worktrees, under `decision:parallel-workspaces/target-selection`,
whose missing-target report is a registered deviation.

## Change

- `crates/core/src/targets.rs` holds `Pattern`, `Selected`, `Missing`, `Selection`, `parse`, `matches`, `select`, and
  `SelectError`, with `Standalones` and `standalones`; `select` applies `env_name::check` to each literal env half, then
  `reach` with the command's `EnvRule`, then the patterns per worktree in merged project order, then the missing test
  once per selected worktree.
- A missing worktree is warned through `Diagnostics::notice` and moved to `Selection.missing`; no git process starts in
  it. Standalones are never selected by a pattern: `standalones` tests each declared standalone once, in merged order,
  warns a missing one with its `standalone/<name>` line, and lists it in `Standalones.missing`.
- The `missing` entry's JSON value and the `target_missing` shape C event are built here for the commands to place.
- `Pattern`, `Selected`, `Missing`, `Selection`, `SelectError`, `Standalones`, `parse`, `matches`, `select`, and
  `standalones` are re-exported by name from `lib.rs` for its tests and for `epic:parallel-workspaces/safe-history`
  ([target-patterns.md](../spec/target-patterns.md), Domain); no command calls them yet.

## Acceptance

- Unit tests of `parse`: `""` is `EmptyPattern`; `a/b/c` is `NotEnvRepo`; `alpha` is `alpha/*`; `/x` and `x/` keep their
  empty half.
- Unit tests of `matches`: `*` never crosses the `/`; `**` matches as `*`; a literal half is exact and case-sensitive.
- Unit tests of `select` over `MemoryFiles`: two patterns matching one worktree give one `Selected`; worktrees come in
  environment order, then merged project order; `projects/x` is refused with `CommandError::EnvName` before anything is
  read from git; an unknown literal env half ends with `UnknownEnv`.
- Unit tests: an absent worktree is `Missing::Absent` and a directory with no `.git` is `NotARepository`, each warned
  once with the spec's line and left out of `worktrees`.
- A component test of `standalones`: an absent standalone and one with no `.git` are each warned once with the spec's
  `standalone/<name>` line, listed in `missing` with their reason, and left out of `present`; a present one is in
  `present` with its path; nothing is read from git.
- A unit test: the `missing` entry and the `target_missing` event serialise as the spec gives for an absent worktree and
  for a missing standalone.
- `cargo dev gate slice` exits 0.
