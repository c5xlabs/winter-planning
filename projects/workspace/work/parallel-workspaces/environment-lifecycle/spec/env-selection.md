# Environment selection

The two rules that list the environments a command's patterns reach, and the refusal of a literal name that names no
environment. `ws destroy` ([destroy.md](./destroy.md), Selection) selects through it, and so does the `<env>/<repo>`
matcher of [target-patterns.md](../../coordinated-git/spec/target-patterns.md) for every worktree command; each links
this leaf and never restates it. The environments present are `present` of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md),
Environments present, which `decision:parallel-workspaces/environment-lifecycle/discovery-authority` keeps; the glob is
`fnmatchcase` and `is_glob` of `crates/core/src/pattern_match.rs`, as built. A literal name reaches this leaf only after
the caller has applied the rule of [env-names.md](./env-names.md) at its own position. Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Domain

```rust
// crates/core/src/env_selection.rs — product:environment-targets, product:target-patterns,
// decision:parallel-workspaces/target-selection, decision:parallel-workspaces/environment-lifecycle/discovery-authority
// Pure but for the file reads of known_env and present; starts no program.

/// Which environments a command's patterns can reach.
pub enum EnvRule {
    Present,   // ws fetch, pull, merge, push, status: the environments present, by index
    Named,     // ws destroy, connect, disconnect, diff, reset, clean, checkout: literal names first, then globbed
               // present ones
}

pub fn known_env(files: &dyn ReadFiles, root: &Path, registry: &[(String, i64)], name: &str) -> bool;

/// The environments `halves` reach under `rule`, in the rule's order, without repeats.
pub fn reach(files: &dyn ReadFiles, root: &Path, config: &WorkspaceConfig, registry: &[(String, i64)],
    halves: &[String], rule: EnvRule) -> Result<Vec<String>, ReachError>;
pub enum ReachError { Selection(SelectionError), Internal(InternalError) }

// crates/core/src/termination.rs — SelectionError gains, beside EmptyPattern and NotBareName:
UnknownEnv { env: String, path: PathBuf }   // path: <root>/<env>; "no environment named <env!r>: none is registered
                                            //  and <path> is not a directory"; registered deviation,
                                            //  decision:parallel-workspaces/target-selection
```

Visibility: `EnvRule`, `known_env`, `reach`, and `ReachError` are `pub` inside their private module and re-exported by
name from `lib.rs`, each module's names on a `pub use` line of their own, with every type their signatures name
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands each tests it
before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`). Everything else here stays `pub(crate)`.

## Interfaces

```text
# reference: modules/workspace/workspace_sync_service.py:768-780, workspace_push_service.py:66-70,
#            workspace_merge_service.py:254-261, workspace_snapshot_service.py:126 (Present);
#            modules/workspace/handlers/workspace_handler.py:350-375, pattern_match.py:48-68 (Named)
halves       the env names a caller passes, duplicates dropped: the PATTERNS of ws destroy, or the env half of each
             <env>/<repo> pattern (target-patterns.md)
literal      a half with none of "*", "?", "[" (pattern_match::is_glob); every other half is a glob
unknown env  first, each literal half, in the order given, must pass known_env; the first that does not ->
             ReachError::Selection(SelectionError::UnknownEnv), and nothing is listed
known_env    name is recorded in the env index registry, or <root>/<name> is a directory, symbolic links followed
             # registered deviation, decision:parallel-workspaces/target-selection: the reference treats the name as
             # matching nothing under Present, and under Named builds the env by name and fails at its first git call
             # with NoSuchPathError, after earlier worktrees were written
Present      the environments present (location-listing.md), in index order; a half reaches each one whose name
             fnmatchcase matches it. A literal half that passes known_env but is not present reaches nothing
Named        the literal halves, sorted by code point; each is reached by name, whether or not it is present
             then the environments present whose name matches any glob half (fnmatchcase(name, half)) and is not a
             literal, sorted by code point
             present is listed only when some half is a glob
present      a candidate directory that cannot be listed -> ReachError::Internal, as present gives it
```
