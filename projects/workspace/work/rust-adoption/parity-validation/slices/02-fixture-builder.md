---
name: fixture-builder
depends-on:
  - scenario-model
---

# Fixture workspace built from a scenario description

Builds `scenario::fixture`: the fixture section of [scenario-format.md](../spec/scenario-format.md) turned into an
isolated tree through the helpers of the
[test harness](../../../parallel-workspaces/component-runtime/spec/test-harness.md). Delivers the build half of
`engineering:scenario-files` under `decision:rust-adoption/parity-validation/scenario-state-prerequisites`.

## Change

- A scenario's `[fixture]`, `files/`, and `dirty/` become a `TempTree` in the spec's build order: remotes, `files/`,
  checkouts, worktrees, the workspace root commit, raw git steps, `dirty/`.
- The token `<ROOT>` inside `files/` and `dirty/` is replaced with the tree's root when copied.
- A fixture entry that names a missing remote or checkout fails before any git call, as the spec's "references" rule
  says.
- The builder applies the fixture builder's resolved check of the spec's Path confinement before every write, clone, and
  git call, and copies a symbolic link under `files/` or `dirty/` as a link. A description the model already rejected
  never reaches the builder.
- A scenario with a `[[stub]]` entry is refused with an error whose subject is `stub`, as the spec's stubs rule says for
  a builder without call capture; nothing is built. The refusal is unconditional: `runner.rs` and its `CALL_CAPTURE`
  constant do not exist yet, and `fixture.rs` reads no such switch, as
  [record-and-replay.md](../spec/record-and-replay.md), Runner, "call capture", says.
- Files touched: `crates/test-support/src/scenario.rs`, `crates/test-support/src/scenario/fixture.rs`,
  `crates/test-support/tests/scenario_fixture.rs`.

## Acceptance

- A description with one remote of two commits, one checkout, one worktree, a workspace root repository, one raw git
  step, and a `dirty/` file builds; the test asserts the worktree's branch, the root repository's single commit, and
  that the `dirty/` file is untracked in the root repository.
- Building that description in two trees yields equal object ids for every ref of every repository.
- A description with only `files/.winter/config.toml` and no `[fixture]` table builds a tree whose `ws/` holds that one
  file byte for byte and no git directory.
- A file under `files/` containing `<ROOT>/remotes/x.git` holds the real root path in the built tree.
- A checkout naming an undeclared remote fails with an error naming the entry, and no directory exists under
  `<ROOT>/ws/projects`.
- A scenario directory whose checkout `path` is `../outside` is malformed when parsed, and no tree is created for it.
- `files/` holds a symbolic link `out` to a directory the test made outside the tree, and a checkout has the `path`
  `out/alpha`: the build fails naming `path`, the clone is not started, and that outside directory is still empty. The
  same holds for a `[[fixture.git]]` entry with `cwd = "out"`: its git call is not made.
- A description with one `[[stub]]` fails with the stub error and creates no file under `<ROOT>/bin`.
- `cargo dev gate slice` exits 0.

## Out of scope

- Stub programs in `<ROOT>/bin`: `program-capture`.
- The runner's own check before each step: `step-runner`.
