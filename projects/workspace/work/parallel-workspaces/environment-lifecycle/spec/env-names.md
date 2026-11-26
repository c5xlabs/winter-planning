# Environment names

The rule every name that addresses a feature environment passes before a command reads git or writes anything, and the
commands that apply it. The rule is a registered deviation:
`decision:parallel-workspaces/environment-lifecycle/env-name-grammar`, `decision:rough-parity`. Reference locations are
paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/env_name.rs — pure: it reads nothing and runs nothing. product:environment-targets
// engineering:worktree-effect-confinement: a name that passes is one directory directly under the root
pub fn check(name: &str, reserved: &Reserved) -> Result<(), EnvNameError>;

// the first components no environment may take, from the loaded configuration
pub struct Reserved { pub standalones: Vec<String> }
impl Reserved {
    pub fn of(config: &WorkspaceConfig) -> Self;   // below, Reserved names
}

// crates/core/src/termination.rs — CommandError gains EnvName(#[from] EnvNameError)
#[derive(Debug, thiserror::Error)]
pub enum EnvNameError {
    #[error("environment name is empty")]
    Empty,
    #[error("environment name {name:?} holds a '/': an environment is one directory under the workspace root")]
    NotOneComponent { name: String },
    #[error("environment name {name:?} starts with '.'")]
    LeadingDot { name: String },                       // ".", "..", and ".winter" included
    #[error("environment name {name:?} is not a valid branch name: {fault}")]
    NotABranchName { name: String, fault: BranchFault },
    #[error("environment name {name:?} is reserved: {location}")]
    Reserved { name: String, location: ReservedLocation },
}
pub enum BranchFault {        // the first that holds, in this order; the text names the fault
    DoubleDot,                // ".."
    Forbidden { c: char },    // a control character (< U+0020 or U+007F), space, ~ ^ : ? * [ \
    AtBrace,                  // "@{"
    LoneAt,                   // the name is "@"
    TrailingDot,              // ends with "."
    LockSuffix,               // ends with ".lock"
    LeadingDash,              // starts with "-"
    Head,                     // the name is "HEAD"
}
pub enum ReservedLocation {
    Workspace,                // "workspace": the workspace service scope
    Projects,                 // "projects": the source checkouts
    Standalone { name: String, path: String },   // the first component of that standalone's location
}
```

Visibility: `check` and `Reserved` with `Reserved::of` are `pub` inside their private module and re-exported by name
from `lib.rs`, on a `pub use` line of their own, with `EnvNameError`, `BranchFault`, and `ReservedLocation`
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands each tests it
before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`).

## Interfaces

### The rule

```text
# decision:parallel-workspaces/environment-lifecycle/env-name-grammar
check(name, reserved), each test in this order, the first failing one the error
  1. name is ""                                          -> Empty
  2. name holds "/"                                      -> NotOneComponent
  3. name starts with "."                                -> LeadingDot
  4. branch name: the tests git applies to refs/heads/<name> under `git check-ref-format --branch <name>`,
     restricted to one component, as git 2.x documents them (git-check-ref-format(1), rules 1, 3-10, and the
     --branch rules)                                     -> NotABranchName, the first BranchFault above that holds
  5. fold(name) equals fold("workspace"), fold("projects"), or fold of the first component of a standalone location
                                                         -> Reserved
fold(s) := the Unicode full case folding of the NFC normalisation of s; only test 5 folds
  # macOS's default volume ignores case and normalisation, so `Projects` or `WORKSPACE` would be projects/ or the
  # workspace scope there; refused on every platform so the rule stays pure (env-name-grammar)
a name is UTF-8 because every argument is (component-runtime/spec/command-surface.md); non-ASCII characters other than
the forbidden ones pass, as git accepts them
no git process runs: the rule is written out here so a refusal comes before any git call
```

### Reserved names

```text
Reserved::of(config)
  standalones   for each loaded standalone entry, in merged order: the first "/"-separated component of its path
                when the path is not empty, else of its resolved name — the location location-listing.md, Locations,
                gives a standalone, relative to the root
  "workspace" and "projects" are always reserved
  location-listing.md = ../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md
```

### Where it applies

```text
# reference: modules/workspace/handlers/init_handler.py:34-38 (refuses `workspace` alone),
#   modules/workspace/pattern_match.py:14-28 (destroy refuses an empty pattern and a "/" alone)
ws init TARGET      after command-surface.md's two ws init rules, before the registry is read:
                    check(TARGET) -> Termination::Command(EnvName), exit 1, nothing read from git, nothing written
                    (init.md)
ws init --all       each name discovery finds: a name that fails is skipped with a warning naming it and the fault,
                    and the run goes on (init.md, Discovery for --all)
ws destroy          after the pattern shape rule: check(each literal PATTERN), the first failing literal in argument
                    order -> Termination::Command(EnvName), exit 1, before any prompt, read of git, or write
                    each name a glob selects: a failing name refuses the command the same way, before the prompt
                    (destroy.md, Selection)
ws index NAME       not applied: lookup records nothing and NAME need not name an environment (index-lookup.md)
ws list             not applied: it lists, and writes nothing
<env>/<repo>        target-patterns.md applies check to the literal env half of every pattern
                    (../../coordinated-git/spec/target-patterns.md); provision and clean apply it to a literal env
                    in their own specs
```

```text
# the reference's results, now refused (registered deviation, decision:parallel-workspaces/environment-lifecycle/env-name-grammar)
ws destroy .           removed the workspace root without a prompt (destroy_service.py:408-411,523-532)
ws destroy ..          removed the parent of the workspace root
ws destroy projects    removed every source checkout
ws init projects       recorded the name, created worktrees inside projects/, and rewired the source checkouts' tracking
ws init a/b            created an environment no ws destroy could address
```
