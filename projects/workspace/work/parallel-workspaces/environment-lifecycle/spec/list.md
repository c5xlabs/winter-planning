# Environment listing

The `ws list` command: the environments present, each with its index, path, and feature branch. Its options are the
`ws list` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands; its
JSON document is the shape A array
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md) names for it,
whose fields this leaf owns. The environments present are those of
[location-listing.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/location-listing.md),
Environments present, which `decision:parallel-workspaces/environment-lifecycle/discovery-authority` keeps; the drift
notice is [drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md)'s. The
command writes nothing. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at
`127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/env_list.rs — product:environment-targets, product:stable-environment-allocation
pub(crate) struct Listed {
    pub name: String,
    pub index: i64,                      // env_index::lookup's answer, registry first
    pub path: PathBuf,                   // <root>/<name>
    pub feature_branch: Option<String>,
    pub distinct_remote_count: u32,
}
pub(crate) struct Listing {
    pub workspace: Workspace,            // as loaded: the root and the values the JSON document reads
    pub environments: Vec<Listed>,       // listing order
}
// the command file's one domain call (context/architecture/changes.md, A command, of winter-workspace): root
// discovery, the load, the drift notice, the registry read, and the findings are its own (Running ws list, steps 1-6),
// as the built locations::list does them
pub(crate) fn list(ports: &Ports<'_>, diagnostics: &mut Diagnostics<'_>, start: &Path)
    -> Result<Listing, EnvListError>;
// the module's own failure, pub and re-exported from lib.rs with the types it carries; the command file maps each
// variant, as it is, to Termination::Config, Termination::Repo, and Termination::Internal
pub enum EnvListError { Config(Box<ConfigError>), Repo(Box<RepoError>), Internal(Box<InternalError>) }
```

## Interfaces

### Running `ws list`

```text
# reference: modules/workspace/handlers/workspace_handler.py:252-270,
#   modules/workspace/internal/read_workspace_repository.py:56-185
order                                             # steps 1-6 are list's; the command file writes step 7
  1. root discovery and the load; a failure is its ConfigError, exit 1
  2. the drift notice (drift-warning.md), written as that leaf gives
  3. the registry read; a refusal is its ConfigError, exit 1, nothing on stdout
  4. the registry findings of allocation.md, as warnings       # product:environment-state-recovery
  5. environments::present (location-listing.md), in its order: ascending index, ties in candidate order
  6. per environment, the branches below; a GitFailure is EnvListError::Repo, or Internal for a git that cannot be
     started, and ends the command, exit 1, nothing on stdout
  7. the result; exit 0
branches(env)                                     # one value per project that is not pinned, in merged order
  l := <root>/<env>/<project>; <l>/.git absent -> None
  head_branch(l) None -> None                     # detached, as init.md's git operations give it
  remote := setting(l, branch.<head>.remote); merge := setting(l, branch.<head>.merge); either None -> None
  remote != "origin", or merge does not start "refs/heads/" -> None
  else merge without "refs/heads/"
  # reads configuration only, so a branch connected and never pushed reads as connected; no network
  # reference: internal/branch_tracking.py:10-54
feature_branch         the first Some of branches(env), or None
distinct_remote_count  the number of distinct Some values of branches(env)
a declared worktree that is absent is None, not an error, and is no warning: ws list reports environments, not
worktrees
```

### JSON document

```text
--json: one array, the environments in listing order; [] when none
element:
  {"environment": {"workspace": {"root_path": <root>, "service_prefix": <string>, "main_branch": <string>,
                                 "base_port": <integer>, "ports_per_env": <integer>},
                   "name": <string>, "index": <integer>, "path": <string>},
   "feature_branch": <string> | null,
   "distinct_remote_count": <integer>,
   "extensions": {}}
service_prefix, main_branch, base_port, ports_per_env   the top-level values of Listing.workspace's configuration
extensions   always an empty object: the reference fills it from in-process environment decorators, which ws list
             never consults (workspace_handler.py:255-261) and the binary does not reproduce (decision:rough-parity)
```

### Text

```text
a table with the headers ENV, FEATURE BRANCH, STATUS; one row per environment in listing order:
  name; feature_branch, or "-" when None; "-"
no environments: the header row alone
layout is the binary's own (output-formats.md, Human output); output::table renders it
```
