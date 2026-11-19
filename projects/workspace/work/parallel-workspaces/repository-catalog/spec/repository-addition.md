# Repository addition

The `repo add` command: the order of its refusals, the block it writes, where in which file the block goes, and what it
reports. Its arguments are the `repo add` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands, and
its refusals refine that file's `repo add` row of Combination rules. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`. How the file is parsed and serialised is the Rust
TOML library's, under `decision:toml-parity`.

## Interfaces

```text
# product:repository-addresses, product:repository-roles, product:edit-repository-inventory
repo add URL [--standalone] [--name TEXT] [--main-branch TEXT] [--ref TEXT] [--git-exclude TEXT]... [--cmd TEXT]...
             [--pinned] [--path TEXT] [--prefix TEXT] [--local] [--json]
kind = standalone with --standalone, else project
order of checks — reference: modules/workspace/command.py:1365-1366, container.py:224-230,705-716,
                  modules/workspace/handlers/repo_handler.py:101-137
  1. workspace root, then the load       configuration-model.md, Root discovery, then Load: its first refusal ends the
                                         command. So a malformed URL outside a workspace is RootNotFound: "refused
                                         before any other check" in command-surface.md means before rules 3 to 7.
  2. URL shape                           CatalogError::InvalidUrl                        URL shape below
  3. kind standalone and --pinned        CatalogError::PinnedStandalone
     kind project and --path             CatalogError::StandaloneOnly { option: Path }
     kind project and --prefix           CatalogError::StandaloneOnly { option: Prefix }
     kind project and --ref              CatalogError::StandaloneOnly { option: Ref }    first of the three that holds
  4. --path given, and absolute or with a ".." component
                                         CatalogError::UnsafePath                        Path rules below
  5. source or name already declared     CatalogError::DuplicateUrl / DuplicateName      Duplicates below
  6. kind standalone: location declared  CatalogError::DuplicatePath                     Duplicates below
  7. the edited file, under the lock     Writing below: 5 and 6 again, then NotArrayOfTables, then the append
an option is "given" when it appears on the command line, with any value, the empty string included
configuration-model.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md
```

Every refusal is a command error, exit 1, with nothing written; a refusal of the load is its `ConfigError`, exit 1.

```text
URL shape — reference: core/internal/click_cli_input_validation_service.py:13-27; command-surface.md, Combination rules
accepted, the whole of URL:
  (http|https|ssh|git)://H[:D][/P][.git][/]          H = one or more of: word character, ".", "+", "-"
  H@H:P[.git]                                        P = one or more of: word character, ".", "/", "+", "~", "-"
                                                     D = one or more decimal digits
word character, digit   Unicode, as Python's \w and \d: a letter or digit of any script, or "_"
end of URL              the match may end before one final "\n", as Python's `$` does; the "\n" then stays part of URL
                        and of everything written and reported from it
not accepted            a user part in the scheme form (ssh://user@host/path), whitespace anywhere else, an empty URL
```

```text
Path rules — reference: repo_handler.py:115-118; decision:parallel-workspaces/repository-catalog/location-policy
absolute           the value starts with "/"
".." component     a component equal to ".." when the value is split at "/", empty components dropped:
                   "a/../b", "a/..", ".." are refused; "..a", "a..b/c" are not
the empty value    accepted; it writes path = "" and the location falls back to the name
```

### Duplicates

```text
# decision:parallel-workspaces/repository-catalog/repository-identities, …/location-policy
new name       --name when given and not empty, else the name derived from URL by step 2 of the effective-name rule
               of repository-removal.md. `--name ""` is still written as name = "" (Block written).
existing       the loaded repositories of the same kind: the shared file's, then the local file's, each in file
               order — configuration-model.md, Layering and Repository entries; an entry the load skips is not one
rule 5         for each existing repository in that order, first the one test, then the other:
                 its url, as a string, equals URL exactly                -> DuplicateUrl { kind, url: URL }
                 its resolved name equals the new name exactly           -> DuplicateName { kind, name: new name }
               an entry without url never matches the first test. A project and a standalone never conflict: the
               same URL or name may be added once as each. `…/api` and `…/api.git` are different URLs.
rule 6         kind standalone only. location = resolve(<root> / (--path when given and not empty, else new name));
               for each declared standalone, in the same order:
                 resolve(<root> / (its path when not empty, else its resolved name)) == location
                                                           -> DuplicatePath { path: --path or new name, as given }
               no other location is compared: not projects/, not .winter/, not a project's location, not a
               location nested inside another standalone
resolve        os.path.realpath with strict=False, as Python's Path.resolve(): components left to right; "." is
               dropped; ".." removes the last component reached so far; a component that is a symbolic link is
               replaced by its target, read relative to its directory and resolved the same way; from the first
               component that does not exist the rest is taken as text. So `--name ..` resolves to the parent
               of the root.
<root>         the root that Root discovery found
```

### Writing

```text
# product:edit-repository-inventory, engineering:add-under-lock
file           without --local   .winter/config.toml
               with --local      .winter/config.local.toml, created when it does not exist
under the lock of configuration-model.md, Write integrity, the edited file is read once and:
  a. the edited file does not exist and it is the shared one         InternalError::Unexpected, exit 1, as the
                                                                       reference's missing-file failure
  b. the edited file is not valid TOML                               ConfigError::FileRead, Toml cause
  c. the load is run again on the edited file as read and the other file as read now; its first refusal is the
     ending. Rules 5 and 6 are checked again on that configuration.   the same errors as rules 5 and 6
  d. the array key of kind is present and holds anything but an array of tables
                                                                       CatalogError::NotArrayOfTables, exit 1
  e. the block is placed (Where the block goes) and the file is replaced once
a to d write nothing. Without a concurrent writer, c finds what rules 1, 5, and 6 found, and the command is the
reference's; with one, of two adds that conflict exactly one appends (decision:visible-gap-policy: hardening).
```

`d` is `decision:parallel-workspaces/repository-catalog/refused-edits`. It is no registered deviation: the reference
tool fails `assert isinstance(existing, AoT)` (`config/internal/write_winter_configuration_repository.py:78-81`) for the
same files, also with exit 1 and nothing written.

### Block written

```text
# reference: config/models.py:204-235,293-310; write_winter_configuration_repository.py:40-48,72-86
array           project_repository  or  standalone_repository, as an array of tables
keys, each only when the condition holds, in the reference's order; nothing else is written
  project       name           --name given
                url            always
                main_branch    --main-branch given
                pinned = true  --pinned given
                git_excludes   one or more --git-exclude, in command-line order
                cmd            one or more --cmd, in command-line order
  standalone    name, url, main_branch as above; then
                path           --path given
                prefix         --prefix given
                ref            --ref given
                git_excludes, cmd as above
values          strings and arrays of strings as given, pinned as the boolean true; their TOML spelling is the
                library's (decision:toml-parity)
```

`--name ""`, `--main-branch ""`, `--path ""`, `--prefix ""`, `--ref ""`, and `--git-exclude ""` are given and are
written, because the reference model drops only unset and default values: it serialises with
`model_dump(exclude_defaults=True, exclude_none=True)`.

### Where the block goes

```text
# decision:toml-parity, engineering:toml-library
model     the edited file's array of kind gains the block as its last element; every other value of the file is
          unchanged; a file that had no such array gets one holding only the block
layout    what toml_edit writes when the block is appended to the parsed document as a new array element; a comment,
          blank line, or key elsewhere in the file is kept as far as toml_edit keeps it
bound     the next load accepts the file and reads it as the model above
```

```text
result
  --json   one Shape A document (../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md,
           Shape per command; this leaf owns its keys, which that leaf's Shape per command parks):
             {"added": true, "kind": "project" | "standalone", "url": <URL as given>}
           no name, file, or layer is reported               reference: repo_handler.py:166-177
  human    one line naming the kind and the URL             reference: repo_handler.py:179
effects    one file created or replaced; nothing else: no clone, fetch, or other git; no setup command; no lock,
           state, or registry write; no directory created; no drift warning
exit       0
```

## Domain

```rust
// crates/core/src/config_edit.rs — pure; beside remove_repository_block
pub(crate) struct NewBlock<'a> {
    pub kind: RepositoryKind,
    pub name: Option<&'a str>, pub url: &'a str, pub main_branch: Option<&'a str>,
    pub pinned: bool,                                        // project only
    pub path: Option<&'a str>, pub prefix: Option<&'a str>, pub r#ref: Option<&'a str>,   // standalone only
    pub git_excludes: &'a [String], pub cmd: &'a [String],
}
// The table of the block, Block written.
pub(crate) fn block_table(block: &NewBlock<'_>) -> toml_edit::Table;
// `document` with the block placed, Where the block goes; Err when the array key holds anything but an array of
// tables. `document` is "" for a file that does not exist.
pub(crate) fn append_repository_block(document: &str, block: &NewBlock<'_>) -> Result<String, NotArrayOfTables>;
pub(crate) struct NotArrayOfTables;

// crates/core/src/repository_catalog.rs
// The checks and the write in the order of Interfaces; `request` holds the parsed arguments as NewBlock does.
pub(crate) fn add(files: &dyn ReadFiles, writer: &dyn WriteFiles, start: &Path, request: &NewBlock<'_>, local: bool)
    -> Result<Added, AddError>;
pub(crate) struct Added { pub kind: RepositoryKind, pub url: String }
pub(crate) enum AddError { Catalog(CatalogError), Config(ConfigError), Internal(InternalError) }
// Catalog -> Termination::Command(CommandError::Catalog), Config -> Termination::Config, Internal ->
// Termination::Internal; each exit 1.

// CatalogError of repository-removal.md, Domain, with the variants repo add adds; the number is the check
pub enum CatalogError {
    // … TargetShape, UnknownKind, NotFound, NotArrayOfTables (step d), Entangled
    #[error("{url}: not a git URL")]
    InvalidUrl { url: String },                                       // 2
    #[error("--pinned applies to project repositories only")]
    PinnedStandalone,                                                 // 3
    #[error("{option} applies to standalone repositories only")]
    StandaloneOnly { option: StandaloneOption },                      // 3
    #[error("{path}: --path is relative and has no .. component")]
    UnsafePath { path: String },                                      // 4
    #[error("a {kind} repository with the URL {url} is declared")]
    DuplicateUrl { kind: RepositoryKind, url: String },               // 5, c
    #[error("a {kind} repository named {name} is declared")]
    DuplicateName { kind: RepositoryKind, name: String },             // 5, c
    #[error("a standalone repository at {path} is declared")]
    DuplicatePath { path: String },                                   // 6, c
}
pub enum StandaloneOption { Path, Prefix, Ref }   // Display: "--path" | "--prefix" | "--ref"
```

The load keeps what these commands read once they land
([configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Load: a key's value is kept once a command reads it):

```rust
// crates/core/src/config.rs — added to Config
pub main_branch: String,                 // top-level main_branch, defaulted: configuration-model.md, Top-level keys
pub projects: Vec<ProjectEntry>,               // merged order: the shared file's entries, then the local file's
pub standalones: Vec<StandaloneEntry>,         // the same; an entry the load skips is in neither
pub(crate) struct ProjectEntry {
    pub name: String,                          // resolved: `name` when not empty, else derived from `url`
    pub url: Option<String>, pub main_branch: Option<String>, pub pinned: bool,
}
pub(crate) struct StandaloneEntry { pub name: String, pub url: Option<String>, pub path: Option<String> }
```

## Open questions

- `resolve` on a symbolic-link loop follows Python's non-strict `realpath`, which leaves the loop unresolved; no
  recorded scenario covers it. Engineering records one with the slice that builds rule 6.
