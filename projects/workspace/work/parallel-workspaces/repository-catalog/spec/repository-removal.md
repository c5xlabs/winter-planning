# Repository removal

The `repo remove` command: which block of which file it removes, and which lines of the user's configuration survive.
Its arguments are the `repo remove` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands. The
grammar of `TARGET` and the three refusals are the `repo remove` row of the same file's Combination rules; this leaf
adds what happens once `TARGET` is well formed.

## Interfaces

```text
repo remove TARGET [--local] [--json]
order of checks
  1. TARGET has no "/"                      refused before the workspace root is looked for: the refusal is the same
                                            inside and outside a workspace
  2. workspace root                         ../../workspace-configuration/spec/configuration-model.md, Root discovery
  3. type is neither project nor standalone refused
  4. the removal below                      not found is refused
file edited   without --local   .winter/config.toml
              with --local      .winter/config.local.toml
              Only that one file is searched. A repository declared in the other file is "not found".
array         type project -> [[project_repository]]      type standalone -> [[standalone_repository]]
              ../../workspace-configuration/spec/configuration-model.md, Repository entries
```

| State of the edited file                                 | Outcome                                                          |
| -------------------------------------------------------- | ---------------------------------------------------------------- |
| `--local` and `.winter/config.local.toml` does not exist | not found; the file is not created; nothing is written           |
| the file has no block of the array                       | not found; nothing is written                                    |
| no block's effective name equals the name                | not found; nothing is written                                    |
| one or more blocks match                                 | the first in file order is removed; the file is replaced; exit 0 |

```text
effective name of a block
# reference: config/internal/write_winter_configuration_repository.py:134-147 — the rule below is that code's, whole
  1. the block has a `name` key  -> the value of `name`, and the URL is not consulted:
       a string, as it is — the empty string included, which is then the effective name
       an integer, in decimal digits
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

### What is removed

```text
# decision:parallel-workspaces/repository-catalog/removal-comment-extent      (registered deviation)
terms, for a block B whose header line is  [[project_repository]]  or  [[standalone_repository]]
  comment line     a line whose first character after optional spaces and tabs is "#"
  blank line       a line holding only spaces and tabs, or nothing
  own comment      of any table header: the unbroken run of comment lines that ends on the line directly before
                   the header line. A blank line, a key line, or another header ends the run; a header whose
                   preceding line is not a comment line has no own comment.
  next header      the first table header line after B's header that is not a sub-table of B; none at end of file
removed — one contiguous range of whole lines
  from   the first line of B's own comment; B's header line when B has none
  to     the line before the own comment of the next header; the line before the next header when it has none;
         the last line of the file when there is no next header
kept, byte for byte
  everything before the range and everything after it: other blocks, top-level keys, comments, blank lines,
  key order, quoting, and the final newline or its absence
blank lines
  blank lines before the start of the range are kept, all of them
  blank lines inside the range — between B's last key and the next header's own comment — are removed, all of them
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
earlier block and nothing above a header as part of the later one. Rows one, four, and five are the deviation, and the
whole of it.

```text
result
  --json   the `repo remove --json` document of ../../component-runtime/spec/output-formats.md, Shape per command;
           "kind" and "name" are TARGET's two parts as given, not values read from the block
  human    one line naming the type and the name
write      one replacement of the one file under ../../workspace-configuration/spec/configuration-model.md,
           Write integrity; no other file changes: not config.lock, not state.toml, no checkout, no worktree
```

## Domain

```rust
// crates/core/src/config_edit.rs — pure text edit; no file access
// Both types are public: CatalogError travels inside the public CommandError, and crates/core/src/lib.rs
// re-exports the two beside it.
#[derive(Debug, Clone, Copy, PartialEq, Eq)]
pub enum RepositoryKind { Project, Standalone }       // Display: "project" | "standalone", TARGET's own words
// Returns the document without the first block of `kind` whose effective name is `name`, or None when no block matches.
pub(crate) fn remove_repository_block(document: &str, kind: RepositoryKind, name: &str) -> Option<String>;

// the module error wrapped by CommandError::Catalog — ../../component-runtime/spec/results-and-exit-codes.md, Domain
#[derive(Debug, thiserror::Error)]
pub enum CatalogError {
    #[error("{target}: a repository is named <type>/<name>")]
    TargetShape { target: String },                       // check 1
    #[error("{kind}: the repository type is project or standalone")]
    UnknownKind { kind: String },                         // check 3
    #[error("no {kind} repository named {name}")]
    NotFound { kind: RepositoryKind, name: String },      // check 4
}
```

The `#[error]` texts show which values a message carries; the wording is the binary's own and is not fixed. Each of the
three ends as `Termination::Command` holding `CommandError::Catalog`, exit 1.

A file that is not valid TOML ends as `ConfigError::FileRead` with the `Toml` cause of
[configuration-model.md](../../workspace-configuration/spec/configuration-model.md), Domain, and is not written.

## Open questions

- The reference column above is derived from reading the reference writer, not from running it; the recorded scenario
  that removes a middle block with comments on both sides confirms or corrects it. Whether a sub-table header inside a
  block (`[project_repository.<key>]`) ends the block in the reference tool has no recorded scenario; engineering
  records one when the rest of this epic's commands are specified.
- The effective name of a block whose `name` is a boolean, a float, a date, an array, or a table, or whose `url` is not
  a string, is the reference tool's text form of that value, which is not confirmed from source and has no recorded
  scenario. [configuration-model.md](../../workspace-configuration/spec/configuration-model.md), Repository entries,
  makes such a value a configuration error once the whole configuration is loaded and validated. Until this epic's
  remaining commands are specified with that load, the binary gives such a block no effective name.
