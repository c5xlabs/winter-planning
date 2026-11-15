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
- Every helper, and every write of the workspace builder, makes the spec's "confinement" check on every path of the call
  before its first effect, through the one check in `tree.rs`; `init`, `clone_remote`, and `add_worktree` refuse the
  tree's root as a destination. Every repository is created with the spec's init command and every clone with its clone
  command. All of it is under `decision:parallel-workspaces/component-runtime/fixture-helper-guarantees`.
- The commit count is held by `TempTree` and advances only once a commit exists, as the spec's "commit dates" says.
- `push_to_remote` follows the three branch cases of the spec.
- `WorkspaceBuilder` writes the `.winter/` files and other files verbatim, creates remotes, project clones, env
  worktrees, and the optional workspace root repository, in the spec's build order.
- No file this change writes spells a name the spec's "names the package never spells" block forbids.
- Files touched: `crates/test-support/src/{lib,tree,git,workspace}.rs`, `crates/test-support/tests/{git,workspace}.rs`.

## Acceptance

- Building the same three-commit remote in two trees yields the same three object ids.
- The spec's fixture "pinned", built as the first thing in a fresh tree, has on `main` exactly the three commit ids the
  test holds as literal constants, oldest first, and its `git config --local --list` holds no `extensions.` key.
- No two commits made by the helpers in one tree share a committer timestamp.
- In a fresh tree, a `commit` with an empty message returns `GitError`, and the next `commit` in that tree carries the
  committer date `2000-01-01T00:00:00+0000`: the failed commit consumed no date.
- A clone's `remote.origin.url` is `https://fixture.invalid/<name>.git`, and `git fetch` in it succeeds with no network:
  the test asserts the fetched ref equals the remote's after `push_to_remote`.
- A workspace whose project names a remote the tree lacks fails to build with the `GitError` of the clone, and the URL
  rule was applied: its stderr contains `<ROOT>/remotes/<name>.git`, with the tree's real root, and does not contain
  `fixture.invalid`. No `projects/<name>/.git` exists afterwards.
- `push_to_remote` with a branch the remote lacks returns an id that is the tip of that branch in the bare repository,
  whose only parent is the tip the remote's default branch had before the call, and the default branch has not moved.
  With no branch named, the returned id is the new tip of the remote's default branch.
- A bare remote built from commits on two branches and one tag has all three refs, `HEAD` on the default branch, and a
  `git config --local --list` with no `remote.` key; that output is equal, byte for byte, for the same remote built in
  two trees, and still has no `remote.` key after `push_to_remote`.
- No file under a built tree's repositories, searched as bytes outside object storage, contains the tree's root path,
  except the two kinds of file the spec's "where the ROOT may appear" names: `gitdir` pointers of worktrees, and
  `FETCH_HEAD`.
- `commit` refuses a repository path before any effect. The test makes a second tree and, in it, a directory holding the
  file `victim.txt`; `commit` on the first tree's `Git` with that directory as the repository and a spec with no branch
  that writes `planted.txt` and deletes `victim.txt` returns `GitError::Unconfined`, and afterwards the directory holds
  `victim.txt` and nothing else. The same spec with the relative repository path `probe-rel` returns
  `GitError::Unconfined`, and no `probe-rel` exists under the test process's working directory.
- A path reached through a symbolic link that leaves the tree is refused. With `<ROOT>/ws/out` a symbolic link to an
  empty directory in a second tree, `init` at `<ROOT>/ws/out/repo`, `clone_remote` to it, and `add_worktree` to it each
  return `GitError::Unconfined`. So does a `commit` in the repository `<ROOT>/ws/repo`, which holds a symbolic link
  `out` to the same directory, with a spec that writes `out/planted.txt`. That directory is still empty afterwards.
- The tree's root is never a repository: `init`, `clone_remote`, and `add_worktree` with the root as destination each
  return `GitError::Unconfined`, and `<ROOT>/.git` does not exist afterwards.
- A workspace built with one remote, one project, and one env worktree has `projects/<name>/.git` as a directory,
  `<env>/<name>/.git` as a file, the worktree on branch `<env>`, and `.winter/config.toml` byte-equal to the input.
- With `root_repository`, the workspace directory is a repository with exactly one commit and `git status --porcelain`
  is empty for files the given `.gitignore` does not exclude.
- A git call that exits non-zero returns `GitError` carrying the argument vector and stderr.
- `cargo dev gate slice` exits 0.

## Out of scope

- Building a fixture from a scenario description: `epic:rust-adoption/parity-validation`.
