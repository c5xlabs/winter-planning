# Drift warning

The warning that the declared projects and the directories under `projects/` disagree. `repo list` writes it, and so do
twelve `ws` commands, whose specs link this leaf for it. It covers projects only; standalones and singletons are never
compared. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`.

## Interfaces

```text
# product:inventory-presence — reference: modules/workspace/drift.py:46-88
declared     the resolved names of the loaded project entries, in merged order (configuration-model.md,
             Repository entries), duplicates included
on disk      when <root>/projects is a directory, symbolic links followed: the names of its entries that are
             directories, symbolic links followed, and whose name does not start with "."; else no names
missing      every declared project whose name is not on disk, sorted by name; a name declared twice is listed twice
undeclared   every name on disk that no declared project has, sorted
sorted       by Unicode code point, as Python sorts strings
no drift     both lists empty: nothing is written
drift        one notice on stderr (../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md,
             Diagnostics on the stderr handle, notice): it names each missing name in order when there is any, then
             each undeclared name in order when there is any; wording and separators are the binary's own
when         after the load and before the command writes its result; under --json as well, never inside the document
listing      a <root>/projects that is a directory and cannot be listed ends the command as InternalError::Unexpected,
             exit 1, as the reference's unhandled OSError; nothing has been written to stdout
writes       nothing to the filesystem; starts no program
configuration-model.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md
```

Reference form, an example under `decision:functional-parity`:

```text
warning: Config and projects/ have drifted — missing from projects/: api, web; undeclared in config: old
```

| Writer                                                                         | Reference call site                                       | Wired by                                         |
| ------------------------------------------------------------------------------ | --------------------------------------------------------- | ------------------------------------------------ |
| `repo list`                                                                    | `modules/workspace/handlers/repo_handler.py:72`           | this epic                                        |
| `ws list`                                                                      | `modules/workspace/handlers/workspace_handler.py:254`     | `epic:parallel-workspaces/environment-lifecycle` |
| `ws connect`, `ws disconnect`, `ws diff`                                       | `workspace_handler.py:316,379,1073`                       | `epic:parallel-workspaces/coordinated-git`       |
| `ws fetch`, `ws pull`, `ws merge`, `ws push`                                   | `workspace_handler.py:676,696,754,779`                    | `epic:parallel-workspaces/coordinated-git`       |
| `ws checkout`, `ws reset`, `ws clean`                                          | `workspace_handler.py:414,494,562`                        | `epic:parallel-workspaces/safe-history`          |
| `ws update`                                                                    | `workspace_handler.py:723`                                | `epic:parallel-workspaces/reproducible-inputs`   |
| `ws status`, `detect` only, no notice: its document marks each missing project | `modules/workspace/workspace_snapshot_service.py:220-232` | `epic:parallel-workspaces/coordinated-git`       |

This epic builds `drift::detect` and `drift::notice_line` with `repo list`, its one writer here. Each other writer is
wired by the epic that builds the command, with the command: its spec links this leaf and says where in its run the
notice is written, and its slice calls the two functions and adds the command's drift scenario. No slice of this epic
touches another epic's command. `repo add`, `repo remove`, and `ws worktrees` do not write it; `ws status` reads
`detect` and writes no notice.

## Domain

```rust
// crates/core/src/drift.rs — pure but for the listing, which goes through the file port
pub(crate) struct Drift { pub missing: Vec<String>, pub undeclared: Vec<String> }
pub(crate) fn detect(files: &dyn ReadFiles, root: &Path, projects: &[ProjectEntry]) -> Result<Drift, InternalError>;
// The notice line of a drift, or None when both lists are empty.
pub(crate) fn notice_line(drift: &Drift) -> Option<String>;
// crates/core/src/files.rs — ReadFiles gains, with this leaf and its siblings, each implemented on OsFiles and on the
// in-memory MemoryFiles: whether a path exists and whether it is a directory, symbolic links followed; the names of
// the entries of a directory, in no fixed order; and resolve, the rule of repository-addition.md, Duplicates.
fn exists(&self, path: &Path) -> bool;
fn is_dir(&self, path: &Path) -> bool;
fn entries(&self, path: &Path) -> io::Result<Vec<OsString>>;
fn resolve(&self, path: &Path) -> PathBuf;
// MemoryFiles holds, per path, a regular file with its bytes, a directory, a symbolic link with its target, or an
// entry that fails every read with an error kind. A parent of an added path is a directory. exists, is_dir, entries,
// and resolve follow links as OsFiles does; entries of an entry that fails reads is that error.
```

A directory name under `projects/` is compared as its bytes, so one that is not valid UTF-8 never equals a declared name
and is undeclared, as in the reference; the notice writes it lossily. The reference compares and sorts Python's
surrogate-escaped text of such a name, an artifact of Python under `decision:functional-parity`.

`ProjectEntry` is [repository-addition.md](./repository-addition.md), Domain.
