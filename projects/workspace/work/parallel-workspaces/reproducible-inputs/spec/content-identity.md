# Content identity

The digest `ws fingerprint` prints for a workspace definition. The digest is contract: both tools give one digest for
one definition (`decision:parallel-workspaces/reproducible-inputs/fingerprint-boundary`). Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`. `RepoMessage` and
`Termination` are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain; shape A is that of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), JSON shapes.

## Domain

```rust
pub struct RepoFingerprint {
    pub name: String,
    pub commit: Sha,       // HEAD; informational, not part of the digest
    pub tree: Sha,         // the tree of the tracked content
    pub dirty: bool,       // a tracked file is staged, modified, deleted, or conflicted; may be true while tree == HEAD^{tree}
}

pub struct WorkspaceFingerprint {
    pub digest: String,    // 64 lowercase hex characters
    pub workspace: RepoFingerprint,
    pub standalones: Vec<RepoFingerprint>,   // ordered by name
}
```

```rust
// engineering:typed-outcomes; RepoMessage (results-and-exit-codes.md, Domain) gains, the wording the reference's
// (workspace_fingerprint_service.py:55):
MissingCheckout { name: String, path: PathBuf }   // "cannot fingerprint <name>: <path> does not exist"; cwd the path
```

## Interfaces

### Git operations

```text
# the program port's Call gains `env: &[(&OsStr, &OsStr)]`, names set over the child environment of child-processes.md
# for that call alone; every other call, the catalog's included, passes an empty slice
# engineering:fingerprint-observation; through the program port of child-processes.md, with the operation spans of
# standalone-pins.md, Operation spans; no network; reference: modules/workspace/internal/gitpython_repository.py:274-349
head(path)          rev-parse HEAD                                         span "git rev-parse"
tree(path)          span "git write-tree"
                    rev-parse --git-path index, without GIT_INDEX_FILE  -> the real index, resolved against path
                    with GIT_INDEX_FILE=<a temporary file outside the repository>:
                      the real index exists -> copied byte for byte, its mtime kept
                      otherwise             -> read-tree HEAD
                      add -u
                      write-tree            -> the tree
                    the real index is never written; blobs and trees left unreferenced in the object store are the
                    only repository writes; the temporary file is removed on every exit path
dirty(path)         span "git status"; with GIT_OPTIONAL_LOCKS=0: status --porcelain --untracked-files=no;
                    dirty iff output is not empty
```

```rust
// crates/core/src/files.rs — the file port gains the scratch copy tree() needs, so no other module names std::fs
pub trait ScratchFiles {
    // A copy of `source` in a new directory under `temp`, bytes and modification time kept; the directory and the
    // copy are removed when the handle drops.
    fn scratch_copy(&self, temp: &Path, source: &Path) -> io::Result<ScratchFile>;
    // A path in a new directory under `temp`, nothing written; removed the same way.
    fn scratch_path(&self, temp: &Path, name: &str) -> io::Result<ScratchFile>;
}
pub struct ScratchFile { /* the path, and the directory removed on drop */ }
// OsFiles alone implements it, naming each directory uniquely as files.rs names its own temporary files; no crate is
// added. `temp` is TMPDIR of the invocation's snapshot, else /tmp, which the fingerprint entry reads from cx.env.
// The Process inputs table of command-surface.md is completed by this row:
//   TMPDIR    The directory ws fingerprint makes its scratch index under; also passed to the child.
```

### `ws fingerprint`

```text
# product:workspace-content-identity; reference: modules/workspace/handlers/fingerprint_handler.py,
#           modules/workspace/workspace_fingerprint_service.py
text      the digest, one line
--json    shape A: {"digest": ..., "workspace": {"name", "commit", "tree", "dirty"}, "standalones": [{...}, ...]}
errors    a checkout path that is not a directory -> RepoError MissingCheckout, exit 1
          a git failure, the workspace root not being a repository included -> that RepoError, exit 1
          each ends as Termination::Repo
          the reference's "requires the implicit workspace repository" refusal is unreachable: its load always
          adds the workspace singleton (config/workspace.py:137-139)
```

The workspace repository is the workspace root, named by its directory's base name.

Inside the identity is the tracked content as `add -u` stages it, so checkout filters apply as Git's staging applies
them and a submodule contributes only its recorded commit. Outside it are untracked files, untracked local
configuration, and the installed component's version. No consistency is claimed for content edited during the
computation.

## Flows

### Digest

```text
# reference: modules/workspace/workspace_fingerprint_service.py:36-69
workspace   := fingerprint(the workspace repository)
standalones := fingerprint(each standalone), sorted by name
preimage    := {"version": 1, "workspace": workspace.tree,
                "standalones": [{"name": s.name, "tree": s.tree} for s in standalones]}
digest      := lowercase hex SHA-256 of the preimage serialised with keys sorted, separators "," and ":", no other
               whitespace, and every non-ASCII character escaped as \uXXXX, a character outside the Basic Multilingual Plane as its
               UTF-16 surrogate pair (Python's json.dumps defaults)
```
