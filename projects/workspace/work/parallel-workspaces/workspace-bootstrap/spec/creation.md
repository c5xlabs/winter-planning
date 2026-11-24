# Creation

The `init` command: creating a workspace in the caller's directory from the installed binary, with no framework clone.
It is a registered deviation, `decision:parallel-workspaces/workspace-bootstrap/creation-ownership`: the reference
tool's `init` is a launcher command that touches an empty `.winter/config.toml` and runs `ws init`, and the command
surface lists it as deliberately absent (`decision:parallel-workspaces/component-runtime/launcher-only-surface`); this
command is the binary's own. The managed blocks are those of [managed-content.md](./managed-content.md); `Termination`,
`CommandError`, and `ConfigError` are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain; shape A is that of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md). Message wording
is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/bootstrap/create.rs — product:create-unforked-workspace
pub struct Created {
    pub root: PathBuf,                    // the caller's directory, as given, not resolved
    pub agents_md: Placement,             // Created | Updated | Unchanged | Conflict (managed-content.md)
    pub claude_md: Shim,                  // Created | Present | Conflict
}
pub enum CreateError {
    Bootstrap(BootstrapError),            // CommandError::Bootstrap, exit 1
    Config(Box<ConfigError>),             // ConfigError::FileWrite, exit 1
    Internal(Box<InternalError>),
}
// crates/core/src/bootstrap.rs — the module's own error, as context/architecture/changes.md, An error, places it;
// termination.rs gains CommandError::Bootstrap(BootstrapError), and lib.rs re-exports BootstrapError. Termination's
// error_type needs no new name: both end as Termination::Command, "Command"
pub enum BootstrapError {
    AlreadyInitialized { path: PathBuf },  // "<path> already exists; run `winter-workspace ws init` to reconcile it"
    JsonNeedsForce,                        // ws prune --json without --force or --dry-run (prune.md)
}
pub fn create(writer: &dyn WriteFiles, files: &dyn ReadFiles, cwd: &Path, version: &str) -> Result<Created, CreateError>;
```

## Interfaces

```text
# decision:parallel-workspaces/workspace-bootstrap/creation-ownership
init [--json]                             a top-level command: commands/init.rs; no argument, no other option
the workspace root is the caller's directory (cx.cwd); no root discovery runs, so a directory inside another
workspace may hold a workspace of its own, as nested workspaces do
order, each after the one before:
  1. <cwd>/.winter/config.toml exists, as anything -> AlreadyInitialized, exit 1, nothing written
  2. create_dir_all(<cwd>/.winter), then one update of <cwd>/.winter/config.toml whose edit writes only when the file
     is absent: a file that appeared since 1 is AlreadyInitialized, nothing written
     content, which parses to exactly these values:
       [versions]
       workspace = "<Identity.version>"            # the agreed pin field, decision:platform/launcher-shim/version-selection
     an io::Error -> ConfigError::FileWrite, exit 1
  3. agents_md::place(cwd)
  4. agents_md::shim(cwd)
  an io::Error in 3 or 4 -> ConfigError::FileWrite naming the file, exit 1; the order is fixed, so the files before
  it in this list were written and stay
no git command runs, nothing is cloned, and no other file is created or changed: .gitignore, seed repositories, and
source-control initialization are the engineer's (product:create-unforked-workspace)
```

The configuration loads as a workspace with no repositories: `versions` is an unknown top-level key, ignored by the load
([configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Unknown keys), and read by Platform alone.

### Result

```text
--json, shape A, exit 0:
  {"root": <cwd>, "config": "created", "agents_md": "created"|"updated"|"unchanged"|"conflict",
   "claude_md": "created"|"present"|"conflict"}
text, exit 0:
  created <cwd>/.winter/config.toml
  AGENTS.md: placed the workspace safety rules | updated them | already current
  CLAUDE.md: created | already imports AGENTS.md
  then: next, read `winter-workspace context setup`, or declare repositories with `winter-workspace repo add` and run
  `winter-workspace ws init`; until the setup topic exists the line names repo add and ws init alone
  (guidance.md, lands)
a conflict is reported, exit 0, and its file is left as it was:
  AGENTS.md conflict   stderr warning naming the file: a malformed safety block, or not UTF-8; the rules were not placed
  CLAUDE.md conflict   stderr warning: CLAUDE.md does not import AGENTS.md; add a line `@AGENTS.md` to it
```

## Flows

```text
empty directory               -> config created, AGENTS.md created (the block alone), CLAUDE.md "@AGENTS.md\n"
directory with AGENTS.md      -> the block prepended above the existing text, with one blank line between; the rest
                                 byte for byte
directory with CLAUDE.md
  holding "@AGENTS.md"        -> left as it is: present
  holding other text          -> left as it is: conflict
directory with .winter/config.toml -> AlreadyInitialized; nothing else read or written
```
