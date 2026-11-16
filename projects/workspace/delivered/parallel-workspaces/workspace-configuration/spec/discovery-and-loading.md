# Discovery and loading

How the binary locates a workspace and turns its two configuration files into one validated value: the modules, the
value a command receives, the seams, and the commands that exercise them. Which values are accepted, coerced, or refused
is [configuration-model.md](./configuration-model.md); the endings and exit codes are
[results-and-exit-codes.md](../../component-runtime/spec/results-and-exit-codes.md); `Context`, `Invocation`, and the
file ports are [runtime.md](../../component-runtime/spec/runtime.md) and
[repository-layout.md](../../component-runtime/spec/repository-layout.md). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`.

## System

The reference tool has no command that shows, validates, or edits configuration, and the binary adds none. The model is
reached through the commands that need a workspace:

| Command                  | Exists | What it exercises                                                                 |
| ------------------------ | ------ | --------------------------------------------------------------------------------- |
| `ws index NAME [--json]` | yes    | Discovery, both files, the merge, every refusal of the model, the state file read |
| `repo remove NAME`       | yes    | The same load before its edit; the replacement of one configuration file          |
| `repo add`, `repo list`  | next   | The repository entries as values; the append writer                               |
| `space KIND`, `ws init`  | later  | `[space]`; the state file writes, the lock, the nested upsert                     |

Every command that needs the workspace loads and validates the whole merged document, so a refusal of any key stops
`ws index` too (`container.py:224-230`). Help, the version, and usage errors never load.

Modules of the `core` crate:

| Module                         | Holds                                                                                                                                                              |
| ------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `workspace_root.rs`            | Root discovery over the read port. Exists.                                                                                                                         |
| `workspace_files.rs`           | The four file names. Exists.                                                                                                                                       |
| `config_file.rs`               | Reads one configuration file into a TOML table.                                                                                                                    |
| `config_merge.rs`              | The merge of the shared and the local table. Pure: no port.                                                                                                        |
| `config_model.rs`              | The configuration types and the build of the typed value from the merged table, in order.                                                                          |
| `config_model/ports.rs`        | `base_port`, `ports_per_env`, `env_aliases`, `envs_per_workspace`, and their range rule.                                                                           |
| `config_model/repositories.rs` | Both repository arrays, the duplicate-name rule, the prefix collision.                                                                                             |
| `config_model/settings.rs`     | Every other top-level key and the `git`, `capabilities`, `keybindings`, `tui`, `core_checks`, `space` tables; the removed keys; the raw `provision` and `service`. |
| `config_model/env_bands.rs`    | `[env]`: the bands, the command entries, the number rendering, the removed `[env.vars]`.                                                                           |
| `config_model/agents.rs`       | `[model_tiers]` and `[agent_model_overrides]`, with the built-in tier table.                                                                                       |
| `config_load.rs`               | The one entry: discovery, two reads, the per-file check, the merge, the build.                                                                                     |

`config_model.rs` also holds what its submodules share to read one value: the `ConfigValue` and the `ValueType` of a
TOML value, the test for an empty form, and the readers of a string and of an array of strings that refuse another type
with `ConfigError::ValueType`. No submodule holds a copy of one.

The reader of two keys, `config_raw.rs`, is removed when `index_lookup` calls `config_load`. The singletons of
[configuration-model.md](./configuration-model.md) need a directory test the read port does not have; they are built
with the first command that lists repositories, in `epic:parallel-workspaces/repository-catalog`.

The state file reader, `state_registry.rs`, exists; `index_lookup` calls it after the load, and its failure is a
configuration error ([lock-and-state.md](./lock-and-state.md),
`decision:parallel-workspaces/workspace-configuration/state-read-failures`).

Libraries: `toml` 1.2 parses, and `toml_edit` 0.25 edits; both are dependencies of the `core` crate already and this
epic adds none. No type of either library appears in a public signature other than inside `TomlParseError`.

## Domain

```rust
// engineering:configuration-snapshot, decision:parallel-workspaces/workspace-configuration/loaded-workspace-value
pub struct Workspace { pub root: PathBuf, pub config: WorkspaceConfig }

pub struct WorkspaceConfig {
    pub service_prefix: String,
    pub main_branch: String,
    pub git_excludes: Vec<String>,
    pub ports: Ports,
    pub adopt_extensions: AdoptExtensions,
    pub prefix: String,
    pub skills_dir: String,
    pub doctor: Option<String>,
    pub lint_scripts: Vec<String>,
    pub git_identity: Option<GitIdentity>,
    pub capabilities: Vec<(String, Vec<String>)>,          // in document order
    pub keybindings: Keybindings,
    pub dashboard_layout: DashboardLayout,
    pub file_size_limits: FileSizeLimits,
    pub env: EnvBands,
    pub space: Space,
    pub model_tiers: Vec<ModelTier>,                       // built-in rows first, then added labels in document order
    pub agent_model_overrides: Vec<(String, AgentOverride)>,
    pub project_repositories: Vec<ProjectRepository>,      // shared entries, then local entries
    pub standalone_repositories: Vec<StandaloneRepository>,
    pub provision: RawTable,                               // not validated at load
    pub service: RawArray,                                 // not validated at load
}

pub struct Ports { pub base_port: i64, pub ports_per_env: i64, pub env_aliases: Vec<String>, pub envs_per_workspace: i64 }
pub enum AdoptExtensions { None, Winter, All }
pub enum DashboardLayout { Auto, ReposAsColumns, ReposAsRows, List }
pub struct GitIdentity { pub name: String, pub email: String }
pub struct Keybindings { pub leader: String, pub timeoutlen: i64, pub bindings: Vec<(String, ConfigValue)> }
pub struct FileSizeLimits { pub injected_bytes: i64, pub reference_bytes: i64 }
pub struct Space { pub root: String, pub kinds: Vec<(String, String)> }

pub struct EnvBands { pub workspace: Vec<EnvVar>, pub feature: Vec<EnvVar>, pub named: Vec<(String, Vec<EnvVar>)> }
pub struct EnvVar { pub key: String, pub value: EnvValue }
pub enum EnvValue { Literal(String), Command(EnvCommand) }
pub struct EnvCommand { pub command: String, pub format: EnvFormat, pub shell: bool, pub exports: Option<Vec<String>> }
pub enum EnvFormat { Raw, Dotenv, Json }

pub struct ModelTier { pub label: String, pub claude: Option<String>, pub codex: Option<String>, pub opencode: Option<String> }
pub enum AgentOverride { Tier(String), Vendors(Vec<(Vendor, VendorModel)>) }
pub enum Vendor { Claude, Codex, Opencode }
pub struct VendorModel { pub model: Option<String>, pub effort: Option<String> }

pub struct RepositoryCommon {
    pub name: String, pub url: Option<String>, pub main_branch: String, pub git_excludes: Vec<String>,
    pub cmd: Vec<String>, pub extension: bool, pub load: Option<LoadMode>, pub entry: Option<Vec<String>>,
}
pub enum LoadMode { Eager, Lazy, None }
pub struct ProjectRepository { pub common: RepositoryCommon, pub pinned: bool, pub nested: Option<Nested> }
pub struct Nested { pub envs: Option<i64>, pub inherit_local: Option<Vec<String>> }
pub struct StandaloneRepository {
    pub common: RepositoryCommon, pub reference: Option<String>, pub path: String, pub prefix: Option<String>,
    pub config_dir: String,
}

pub struct RawTable(/* a parsed TOML table */);
pub struct RawArray(/* parsed TOML values */);
```

`keybindings.bindings` values keep their TOML form; their rendering as key specs belongs to
`epic:agent-ready-operation/workspace-dashboard`. `Space` holds the strings as written; resolving `~` reads `HOME` and
belongs to `epic:agent-ready-operation/artifact-space`. These types are named by command files, so the facade exports
them under the facade rule of [repository-layout.md](../../component-runtime/spec/repository-layout.md); a field no
command reads yet is not unused code, because the type is public.

## Interfaces

```rust
// config_load.rs — the only entry a domain module calls
pub fn load(files: &dyn ReadFiles, start: &Path) -> Result<Workspace, ConfigError>;

// workspace_root.rs — exists
pub fn find(files: &dyn ReadFiles, start: &Path) -> Result<PathBuf, ConfigError>;

// config_file.rs
pub(crate) fn read(files: &dyn ReadFiles, path: &Path) -> Result<toml::Table, ConfigError>;

// config_merge.rs
pub(crate) fn merge(shared: toml::Table, local: toml::Table) -> Result<toml::Table, ConfigError>;

// config_model.rs
pub(crate) fn build(merged: &toml::Table) -> Result<WorkspaceConfig, ConfigError>;

// state_registry.rs — exists; the result type changes under decision:…/state-read-failures
pub(crate) fn read(files: &dyn ReadFiles, root: &Path) -> Result<BTreeMap<String, i64>, ConfigError>;
```

```text
# decision:parallel-workspaces/workspace-configuration/loaded-workspace-value
Context, Invocation      no new field; none of the four places that build an Invocation changes
who loads                a domain entry, once, from (cx.files, cx.cwd); it passes &Workspace to what it calls
who never loads          a command file; the composition root; anything before a command is selected
cache                    none: one command has one domain entry
writers                  take the root from the Workspace and read the file again under the lock; write-path.md
```

```text
# decision:parallel-workspaces/workspace-configuration/domain-reporting
loading                  writes no log record and no notice; the reference tool's loader has no logging call
a domain entry that reports, in a later epic:
                         takes `&mut Diagnostics<'_>`, a borrow of cx.err and cx.log made by its command file,
                         and writes through diagnostics::record and diagnostics::notice of runtime.md
                         Diagnostics is built with the first entry that needs it; it is not a port and no
                         Invocation carries it
```

Process inputs of discovery and loading: the working directory, as `Invocation.cwd` gives it, and the bytes of the two
files. No environment variable, no argument, and no clock is read; the list of process inputs is in
[command-surface.md](../../component-runtime/spec/command-surface.md).

## Flows

```text
# engineering:configuration-snapshot, engineering:inert-configuration-read
load(files, start):
    root   = workspace_root::find(files, start)?                 # configuration-model.md, Root discovery
    shared = config_file::read(files, root/.winter/config.toml)?
    local  = config_file::read(files, root/.winter/config.local.toml)?
    agent_model_overrides present and not a table in shared, then in local -> AgentOverridesNotTable
    project_repository, then standalone_repository, present and not an array,
                                          in shared, then in local -> RepositoriesNotArray
    merged = config_merge::merge(shared, local)?                 # configuration-model.md, Layering
    config = config_model::build(&merged)?                       # configuration-model.md, Load, steps 1 to 9
    Workspace { root, config }

config_file::read(files, path):
    files.is_file(path) is false        -> an empty table        # missing, a directory, a dangling link
    files.read(path) fails              -> FileRead { path, cause: Io }
    the bytes are not UTF-8             -> FileRead { path, cause: Toml }
    the text is not valid TOML          -> FileRead { path, cause: Toml }

index_lookup::answer(files, start, name):
    workspace = config_load::load(files, start)?
    registry  = state_registry::read(files, &workspace.root)?    # lock-and-state.md, `.winter/state.toml`
    the lookup of lock-and-state.md over workspace.config.ports and the registry
```

| Failure                                   | Ending                                                                                       | Exit |
| ----------------------------------------- | -------------------------------------------------------------------------------------------- | ---- |
| No root at or above the working directory | Configuration error, `RootNotFound`                                                          | 1    |
| A file cannot be read or parsed           | Configuration error, `FileRead`                                                              | 1    |
| The state file is refused when it is read | Configuration error, `FileRead`; the cases are in [lock-and-state.md](./lock-and-state.md)   | 1    |
| Any refusal of the model                  | Configuration error, the variant named in [configuration-model.md](./configuration-model.md) | 1    |
| A configuration file cannot be replaced   | Configuration error, `FileWrite`                                                             | 1    |

`repo remove` loads after the shape of its target is checked, in the place of root discovery, and before its type word
is checked and its edit is made, so an invalid configuration refuses the removal and leaves both files as they were,
whichever of the two files is at fault and whichever the removal edits. That the reference tool does the same follows
from its wiring (`container.py:714-722`) and is to be confirmed by a probe: the recorded scenarios
`repo-remove/invalid-configuration` and `repo-remove/invalid-local-configuration`.

## Data

Where the two parsers may differ. The binary reads each text as its parser does, and the difference is one registered
deviation (`decision:parallel-workspaces/workspace-configuration/toml-dialect`):

| Text                                                 | Reference tool, TOML 1.0 | Binary, `toml` 1.2.1       |
| ---------------------------------------------------- | ------------------------ | -------------------------- |
| An integer outside the signed 64-bit range           | accepted                 | to be confirmed by a probe |
| A newline or a trailing comma inside an inline table | refused                  | to be confirmed by a probe |
| The `\e` and `\xHH` string escapes                   | refused                  | to be confirmed by a probe |
| A time without seconds                               | refused                  | to be confirmed by a probe |

The probe is one unit test of `config_file` that feeds each text to the parser and asserts whether it is accepted. The
Binary column is what that probe finds, and the wording of the deviation row in
[command-surface.md](../../component-runtime/spec/command-surface.md) is confirmed by it. The Reference tool column is
TOML 1.0 as its two parsers are documented to read it; its first row is to be confirmed by a probe, the recorded
scenario `ws-index/config-integer-beyond-64-bits`, and the other rows are recorded by no scenario of this epic.

## Open questions

- That the `Display` of a `toml` value yields its TOML text, which `ConfigValue.text` holds, is to be confirmed by a
  probe: a unit test of `config_model.rs` formatting one value of each type.
- That the shortest digits of a float come from the standard formatting of the language, in both plain and exponent
  form, is to be confirmed by a probe: the unit test table of the number rendering in `config_model/env_bands.rs`.
- That an unhandled error of the reference tool exits 1 is to be confirmed by a probe: the recorded scenario
  `ws-index/config-not-utf8`.
