---
name: git-fixtures
depends-on:
  - process-harness
---

# Git fixture helpers and the workspace builder

Builds the `git.rs` and `workspace.rs` sections of [test-harness.md](../spec/test-harness.md): deterministic
repositories, bare remotes reached through the fixture URL rule, and a fixture workspace assembled from literal inputs.
Completes the shared test support `engineering:testable-boundaries` requires and the owned remotes of
`engineering:rust-adoption/parity-validation/disposable-runtime-proof`, under
`decision:parallel-workspaces/component-runtime/harness-provenance`.

## Change

- `Git` runs git through `invoke` under the tree's pinned environment with a deadline and offers `new`, `run`, `init`,
  `commit`, `bare_remote`, `clone_remote`, `add_worktree`, and `push_to_remote` with the commit-date rule of the spec.
- `bare_remote` follows the spec's four steps: the bare repository is initialised and pushed to by path, never cloned,
  so it has no remote and its config holds no path.
- `WorkspaceBuilder` writes the `.winter/` files and other files verbatim, creates remotes, project clones, env
  worktrees, and the optional workspace root repository, in the spec's build order.
- No file this change writes spells a name the spec's "names the package never spells" block forbids.
- Files touched: `crates/test-support/src/{lib,git,workspace}.rs`, `crates/test-support/tests/{git,workspace}.rs`.

## Acceptance

- Building the same three-commit remote in two trees yields the same three object ids.
- No two commits made by the helpers in one tree share a committer timestamp.
- A clone's `remote.origin.url` is `https://fixture.invalid/<name>.git`, and `git fetch` in it succeeds with no network:
  the test asserts the fetched ref equals the remote's after `push_to_remote`.
- A bare remote built from commits on two branches and one tag has all three refs, `HEAD` on the default branch, and a
  `git config --local --list` with no `remote.` key; that output is equal, byte for byte, for the same remote built in
  two trees, and still has no `remote.` key after `push_to_remote`.
- No file under a built tree's repositories, searched as bytes outside object storage, contains the tree's root path,
  except the two kinds of file the spec's "where the ROOT may appear" names: `gitdir` pointers of worktrees, and
  `FETCH_HEAD`.
- A workspace built with one remote, one project, and one env worktree has `projects/<name>/.git` as a directory,
  `<env>/<name>/.git` as a file, the worktree on branch `<env>`, and `.winter/config.toml` byte-equal to the input.
- With `root_repository`, the workspace directory is a repository with exactly one commit and `git status --porcelain`
  is empty for files the given `.gitignore` does not exclude.
- A git call that exits non-zero returns `GitError` carrying the argument vector and stderr.
- `cargo dev gate slice` exits 0.

## Out of scope

- Building a fixture from a scenario description: `epic:rust-adoption/parity-validation`.
