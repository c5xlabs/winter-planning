# Repository removal

The `repo remove` command: which block of which file it removes, and which lines of the user's configuration survive.
Its arguments are the `repo remove` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands. The
grammar of `TARGET` and the three refusals are the `repo remove` row of the same file's Combination rules; this leaf
adds what happens once `TARGET` is well formed, and owns the command's `--json` document.

## Interfaces

```text
# product:edit-repository-inventory
repo remove TARGET [--local] [--json]
order of checks
  1. TARGET has no "/"                      CatalogError::TargetShape, before the workspace root is looked for: the
                                            refusal is the same inside and outside a workspace
  2. workspace root, then the load          configuration-model.md, Root discovery, then Load: its first refusal ends
                                            the command and nothing is written
  3. type is neither project nor standalone CatalogError::UnknownKind
  4. the edited file, under the write lock  the table below
file edited   without --local   .winter/config.toml
              with --local      .winter/config.local.toml
              Only that one file is searched. A repository declared in the other file is "not found".
array         type project -> project_repository      type standalone -> standalone_repository
              configuration-model.md, Repository entries
configuration-model.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md
```

The edited file is read once, under the lock of
[write-path.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/write-path.md), and its rows are
checked in this order; the first that holds decides. The edit works on those bytes, never on the loaded value, so an
entry another writer added since the load is kept.

| State of the edited file, as read under the lock                                     | Outcome                                                                 |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------------------- |
| `--local` and `.winter/config.local.toml` does not exist                             | `NotFound`; the file is not created                                     |
| not valid TOML                                                                       | `ConfigError::FileRead` with the `Toml` cause; the file is not written  |
| the array key is absent                                                              | `NotFound`                                                              |
| the array key holds anything but an array of tables                                  | `NotArrayOfTables`                                                      |
| no block's effective name equals the name                                            | `NotFound`                                                              |
| the text with the lines below removed does not parse to the document less that block | `Entangled`                                                             |
| otherwise                                                                            | the first matching block in file order is removed; the file is replaced |
| the replacement fails                                                                | `ConfigError::FileWrite` naming the file; the file is as it was         |

Every row but `otherwise` writes nothing and exits 1. A failed replacement is
`decision:parallel-workspaces/workspace-configuration/write-failure-kind`. Two of the refusals are
`decision:parallel-workspaces/repository-catalog/refused-edits` (registered deviations): `NotArrayOfTables` for an
inline array of tables holding a matching element, which the reference tool removes
(`config/internal/write_winter_configuration_repository.py:93-100`), and `Entangled`, where the reference removes the
block. For an empty inline array the reference also reports not found and writes nothing.

```text
which values reach the NotArrayOfTables row
  an array written inline          project_repository = [{ url = "…" }], = [], = [1]   the load accepts an array and
                                                                                        skips its non-table elements
  any value that is not an array   a string, a boolean, a number, a table               refused at check 2 as
                                   ConfigError::RepositoriesNotArray naming the key and the file, exit 1
                                   (decision:parallel-workspaces/workspace-configuration/repository-array-type);
                                   it reaches this row only when another writer wrote it between the load and the lock
```

```text
effective name of a block
# reference: config/internal/write_winter_configuration_repository.py:134-147
  1. the block has a `name` key  -> its value decides, and the URL is not consulted:
       a string        as it is — the empty string included, which is then the effective name
       an integer      its decimal digits
       any other value no effective name
  2. else the block has a `url` key whose value is not the empty string -> derived from the URL text:
       a. strip every trailing "/" character
       b. cut = the later of the last "/" and the last ":" in what is left; take the text after cut,
          or all of it when it holds neither character
       c. drop one trailing ".git", when present
  3. else -> the block has no effective name and never matches
examples      https://host/org/api.git -> api      https://host/org/api/ -> api      host:api.git -> api
              git@host:org/api.git -> api          host/a:b.git -> b                 https://host/ -> host
comparison    exact, case-sensitive, against the name part of TARGET (everything after TARGET's first "/")
```

The load refuses a `name` or `url` that is not a string on every block it keeps
([configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Value types), so a non-string value reaches this rule only on a block the load skips because its `name` and `url` both
read as empty. The reference tool matches such a block by Python's text of the value (`False`, `[]`); that text is an
artifact of Python under `decision:functional-parity`, and the binary gives the block no name.

### What is removed

```text
# decision:parallel-workspaces/repository-catalog/removal-comment-extent      (registered deviation)
terms, for a block B whose header line is  [[project_repository]]  or  [[standalone_repository]]
  comment line     a line whose first character after optional spaces and tabs is "#"; a line inside a multi-line
                   string is none
  blank line       a line holding only spaces and tabs, or nothing
  own comment      of any table header: the unbroken run of comment lines that ends on the line directly before
                   the header line. A blank line, a key line, or another header ends the run; a header whose
                   preceding line is not a comment line has no own comment.
  sub-table of B   a table header whose table the parsed document places inside B: [project_repository.<key>] or
                   [[project_repository.<key>]] after B and before the array's next block, wherever it stands
  next header      of a line: the first table header line after it that is not a sub-table of B; none at end of file
  end of a range   the line before the next header's own comment; the line before the next header when it has none;
                   the last line of the file when there is no next header
removed — whole lines, in one or more ranges
  main range       from the first line of B's own comment, or B's header line when B has none, to the end of a range
                   after B's header
  sub-table range  for each sub-table of B whose header line lies outside every range so far: from that header line
                   to the end of a range after it; the comment lines directly above that header are kept
kept, byte for byte
  everything outside the ranges: other blocks, top-level keys, comments, blank lines, key order, quoting, line
  endings, a byte order mark, and the final newline or its absence
blank lines
  blank lines before the start of a range are kept, all of them
  blank lines inside a range — between B's last key and the next header's own comment — are removed, all of them
  nothing is inserted and nothing is collapsed: when B was the last block, the blank line that separated it from
  the block before now ends the file
comment lines after B's last key that are not the next header's own comment (a blank line separates them from
the next header, or there is no next header) are inside the range and removed
```

| Layout around B                                                                 | Reference tool                        | Binary                                       |
| ------------------------------------------------------------------------------- | ------------------------------------- | -------------------------------------------- |
| a comment run directly above B's header                                         | keeps it                              | removes it with B                            |
| a comment run, a blank line, then B's header                                    | keeps it                              | keeps it: it is not B's own                  |
| after B's last key: comment lines, a blank line, then the next header           | removes them with B                   | the same                                     |
| after B's last key: comment lines, a blank line, comment lines, the next header | removes all of them with B            | keeps the run directly above the next header |
| after B's last key: comment lines directly followed by the next header          | removes them with B                   | keeps them: the next header's own            |
| neither B nor the next header has an own comment                                | removes B's header to the next header | the same                                     |

The reference column is the reference writer's behaviour: it treats every line between two headers as part of the
earlier block and nothing above a header as part of the later one. Rows one, four, and five are
`decision:parallel-workspaces/repository-catalog/removal-comment-extent`, and the whole of it. Rows one and four are
confirmed by the recorded scenario `scenarios/repo-remove/removes-middle-block` of `winter-workspace` (reference
`b57b020`), whose record keeps `# second: web` above the removed block and removes `# after web`, the blank line, and
`# third: docs`.

```text
result
  --json   one Shape A document (../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md,
           Shape per command; this leaf owns its keys, which that leaf's Shape per command parks):
             {"removed": true, "kind": "project" | "standalone", "name": <string>}
           "kind" and "name" are TARGET's two parts as given, not values read from the block
           reference: modules/workspace/handlers/repo_handler.py:192-203
  human    one line naming the type and the name                         reference: repo_handler.py:205
write      one replacement of the one file through WriteFiles::update (write-path.md); no other file changes:
           not config.lock, not state.toml, no checkout, no worktree, no branch; no git is run; no drift warning
exit       0
```

## Domain

```rust
// crates/core/src/config_edit.rs — pure text edit; no file access
// RepositoryKind and CatalogError are public: CatalogError travels inside the public CommandError, and
// crates/core/src/lib.rs re-exports the two beside it.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum RepositoryKind { Project, Standalone }       // Display: "project" | "standalone", TARGET's own words

pub(crate) struct BlockRemoval {
    pub(crate) text: String,       // the document without the block
    pub(crate) index: usize,       // the block's position among the blocks of its kind in the text it came from, from 0
}
// The document without the first block of `kind` whose effective name is `name`; None when no block matches, when
// the array is not an array of tables, and when the document does not parse.
pub(crate) fn remove_repository_block(document: &str, kind: RepositoryKind, name: &str) -> Option<BlockRemoval>;
// Whether `edited` is `original` less the block of `kind` at `index`: both parse, every other key holds the same
// value, and the remaining blocks are the others in order. The Entangled row asks it.
pub(crate) fn lacks_one_block(original: &str, edited: &str, kind: RepositoryKind, index: usize) -> bool;

// the module error wrapped by CommandError::Catalog —
// ../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md, Domain.
// repository-addition.md, Domain, adds the variants of repo add to the same enum.
#[derive(Debug, thiserror::Error)]
pub enum CatalogError {
    #[error("{target}: a repository is named <type>/<name>")]
    TargetShape { target: String },                                   // check 1
    #[error("{kind}: the repository type is project or standalone")]
    UnknownKind { kind: String },                                     // check 3
    #[error("no {kind} repository named {name}")]
    NotFound { kind: RepositoryKind, name: String },                  // check 4
    #[error("the {kind} repositories of {} are not written as blocks", .path.display())]
    NotArrayOfTables { kind: RepositoryKind, path: PathBuf },         // check 4; repo add as well
    #[error("the {kind} repository {name} cannot be removed without changing other entries")]
    Entangled { kind: RepositoryKind, name: String },                 // check 4
    // … repo add's variants
}
```

The `#[error]` texts show which values a message carries; the wording is the binary's own and is not fixed. Each variant
ends as `Termination::Command` holding `CommandError::Catalog`, exit 1. A file that is not valid TOML ends as
`ConfigError::FileRead` with the `Toml` cause of
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Domain, and is not written.
