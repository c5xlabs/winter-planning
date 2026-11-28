# Prune

The `ws prune` command: finding material Workspace once managed and no configuration declares now, retaining what could
hold work, and removing the rest. Its options are the `ws prune` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands; its
JSON document is the one
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md) holds for it,
which this leaf now owns; the prompt is the confirmation of
[runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md), Confirmation. The workspace
exclude file and `finalize` are those of [managed-content.md](./managed-content.md); the events after a removal are
those of [reconciliation.md](./reconciliation.md). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/prune.rs — product:prune-managed-material, engineering:prune-ownership-evidence
pub enum OrphanKind { ProjectClone, StandaloneClone, BrokenSymlink, OrphanAgentCopy }
// "project_clone", "standalone_clone", "broken_symlink", "orphan_agent_copy"
pub struct Orphan {
    pub kind: OrphanKind,
    pub path: PathBuf,                    // absolute, under the root
    pub block: Option<String>,            // the exclude block that named it; None for a project clone
    pub retained: Option<Retained>,       // None: safe to remove
    pub scanned: Scanned,                 // what the recheck of remove compares against
}
pub struct Scanned { pub parent: PathBuf, pub entry: Entry }   // parent resolved at scan; the entry's own type
pub enum Entry { Directory, Link, File }  // not following the last component
pub enum Retained {                       // first that holds wins, in this order; the notes text
    OutsideRoot,                          // "resolves outside the workspace"
    NotAClone,                            // "not a git clone (delete by hand if intentional)"
    LinkedWorktree,                       // "is a linked worktree (.git is a file)"
    HasWorktrees,                         // "has linked worktrees"
    Dirty,                                // "uncommitted or untracked changes"; also an unreadable status
    LocalOnly,                            // "has commits no remote-tracking branch holds"; also unreadable history
    Stashes,                              // "has stashes"; also an unreadable stash list
}
pub struct Scan { pub orphans: Vec<Orphan>, pub evidence: Evidence }
pub enum Evidence { Read, Unreadable }    // Unreadable: the exclude file could not be located or read
pub enum Removal { Removed, Skipped(Retained), Error(String) }      // "removed" | "skipped" | "error"
pub fn scan(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, workspace: &Workspace) -> Scan;
pub fn remove(ports: &Ports<'_>, workspace: &Workspace, orphan: &Orphan) -> Removal;
pub fn reaggregate(ports: &Ports<'_>, workspace: &Workspace, retained_blocks: &[String]) -> Result<Finalized, GitFailure>;
```

## Interfaces

### Candidates

```text
# reference: modules/workspace/prune_service.py:64-279; read-only: the exclude file comes from locate, never
# locate_for_write, and no git configuration is written by a scan or a dry run
eligible names := every declared standalone's name, and every project that is an extension (extension-management's
                  term: `extension` not false and projects/<name>/winter-ext.toml a file)
project_clone     each directory entry of <root>/projects that is a directory or a symbolic link to one, named by no
                  declared project, sorted; the path is the entry's own, not resolved
standalone_clone  for each block of blocks(exclude file) whose name holds no "/" and is not eligible, in file order:
                  each line, trimmed, matching ^/(.+?)/?$ whose group is not "projects" and does not start ".claude/":
                    p := resolve(<root>/<group>); kept when it exists and was not already a candidate
broken_symlink    each entry of <root>/.claude/skills, <root>/.claude/agents, <root>/.codex/skills that is a symbolic
                  link whose target does not exist, sorted per directory
orphan_agent_copy for each such non-eligible block, each line matching ^([^/]+/[^/]+)/(.+)-\*$ whose first group is a
                  vendor agent directory (.claude/agents, .codex/agents, .opencode/agent): each regular file of
                  <root>/<group 1> whose name starts "<group 2>-", sorted, once
exclude file      locate(root); NotARepository -> no blocks, Evidence::Read; a GitFailure or an unreadable file
                  -> no standalone_clone and no orphan_agent_copy candidates, Evidence::Unreadable, and a warning
                  naming the cause
```

### Retention

```text
# decision:parallel-workspaces/workspace-bootstrap/prune-retention; registered deviations: the reference treats a
# standalone path with no .git as safe, checks only the .git presence, linked worktrees, and status
retained(c) for project_clone and standalone_clone; broken_symlink and orphan_agent_copy are never retained
  c.path is the root, or not under it                     -> OutsideRoot
  <path>/.git absent                                      -> NotAClone
  <path>/.git a file                                      -> LinkedWorktree
  <path>/.git/worktrees a non-empty directory             -> HasWorktrees
  git::clean(path) false                                  -> Dirty            # untracked files count; a failure reads dirty
  git rev-list --max-count=1 HEAD --branches --tags --not --remotes   prints a commit, or fails -> LocalOnly
  git rev-parse --verify --quiet refs/stash               exit 0, or any exit but 1 -> Stashes
  otherwise None
```

### Running `ws prune`

```text
# reference: modules/workspace/handlers/workspace_handler.py:960-1052
order
  1. root discovery and the load; a failure is its ConfigError, exit 1
  2. --json without --force and without --dry-run -> BootstrapError::JsonNeedsForce, exit 1, nothing read or written
     # registered deviation, prune-retention: the reference removes without asking under --json
  3. scan
text (no --json)
  4. no orphans -> "Nothing to prune. Workspace is clean."; exit 0
  5. the preview, one line per orphan: "<' ' safe, '!' retained> <kind, padded to 18> <path>   [<evidence>]", then
     "  (<notes>)" when retained; evidence: "projects/" for a project_clone, "block <name>" for a standalone_clone or
     orphan_agent_copy, "dangling link" for a broken_symlink (product:prune-managed-material: ownership evidence)
  6. --dry-run -> stop; exit 0
  7. without --force, none safe -> "Nothing to remove (all orphans are blocked). Resolve the notes above and re-run.";
     exit 0
  8. without --force: confirm "Remove <n> orphan(s)?", n the safe ones; declined or end of input -> Aborted, exit 1,
     and every orphan is unattempted: nothing is removed and the notice says the run was aborted
  9. each orphan, in scan order: remove, then its line "  remove <rel>", "  skip   <rel> (<notes>)", or
     "  error  <rel> (<cause>)", rel relative to the root. A line that cannot be written stops the removals: the
     orphans not yet reached are unattempted and left as they are, and the invocation ends with the write's
     Termination (product:prune-managed-material: removed, retained, and unattempted)
 10. any Removed and Evidence::Read -> reaggregate; its RepoAction or RepoError line through bootstrap::text
     (reconciliation.md); exit 0, whatever a removal reported
--json (with --dry-run or --force; step 2 refused the rest)
  no prompt, no preview text; the orphans in scan order
  --dry-run -> {"dry_run": true, "orphans": [... action "would_remove" or "skipped"]}; exit 0, also with no orphans
  --force   -> remove each, reaggregate as in 10, its shape C events first (output-formats.md, JSON-mode departures),
               then {"dry_run": false, "orphans": [... action "removed" | "skipped" | "error"], "excludes_updated"};
               exit 0, also with no orphans, where excludes_updated is false
remove(orphan)                                            # engineering:prune-ownership-evidence
  retained at scan -> Skipped
  recheck, for every kind, now:
           resolve(parent of path) is scanned.parent and lies under resolve(root), and the last
           component is the same entry: for a clone the entry type the scan recorded, for a link still a link whose
           target is missing, for a copy still a regular file -> otherwise Skipped, note "changed since the scan"
           for a clone, retained(c) again -> a reason that now holds is Skipped with it
  WriteFiles::remove(path); an io::Error -> Error(<cause>), the path partly removed and left as it is
  -> Removed
reaggregate(retained_blocks)                              # decision:parallel-workspaces/workspace-bootstrap/prune-retention: ownership stays with retained data
  finalize(mode, eligible, keep: retained_blocks), eligible as reconciliation.md's standalone target builds it for the
  current configuration; retained_blocks := the block of every standalone_clone or orphan_agent_copy that was retained,
  skipped, or failed to be removed, so its block and its lines stay as they are
lock and state records are not changed by prune
```

### JSON document

```text
{"dry_run": bool, "orphans": [orphan...], "excludes_updated": bool}     # excludes_updated only when dry_run is false
orphan: {"kind", "path", "safe_to_remove", "notes", "action"} plus "error" when action is "error"
  notes: "" when safe; else the Retained text, or the recheck's
  action: "would_remove" | "skipped" under --dry-run; "removed" | "skipped" | "error" otherwise
excludes_updated: reaggregate ran and returned Ok, whatever its Finalized: the reference reports finalize's success,
  true also when nothing changed or the root is no repository; false when nothing was removed, when the evidence was
  Unreadable, or when reaggregate failed
```
