# Configuration model

The workspace configuration files, their keys, layering, validation, and writers. The binary reads and writes the
reference tool's files unchanged (`decision:configuration-transition`). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `legacy-winter` repository at `b57b020`. The lock and machine-state files are
[lock-and-state.md](./lock-and-state.md); error rendering is
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md).

Under `decision:functional-parity` the contract is which values are accepted, coerced, defaulted, or refused, the error
kind of each refusal with the values it carries, and the bytes written to the files. Which fault a step names when it
meets several is outside the contract, under `decision:parallel-workspaces/workspace-configuration/within-step-order`.
The wording of a configuration error is the binary's own; the reference texts are in `config/workspace.py`.

## System

| File                        | Format | Holds                            | Committed | Written by                                                                                                             |
| --------------------------- | ------ | -------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------- |
| `.winter/config.toml`       | TOML   | Shared workspace declaration     | yes       | `repo add`, `repo remove`; `ws update` when it writes a standalone `ref`                                               |
| `.winter/config.local.toml` | TOML   | Machine-local overlay            | no        | `repo add --local`, `repo remove --local`; the `ref` fallback; nested-workspace delegation by an outer `ws init <env>` |
| `.winter/config.lock`       | TOML   | Resolved standalone pins         | yes       | `ws init`, `ws pull`, `ws update`                                                                                      |
| `.winter/state.toml`        | TOML   | Machine-local env index registry | no        | `ws init` (assign), `ws destroy` (remove)                                                                              |

```text
# decision:parallel-workspaces/workspace-configuration/layering-policy
layers, low to high:   .winter/config.toml  <  .winter/config.local.toml
there is no user-level or global configuration file
never loaded or executed: .winter/plugins/<name>/, ~/.config/winter/plugins/<name>/, <standalone repo>/plugin.py
                          # decision:agent-ready-operation/workspace-dashboard/contribution-scope
```

Singletons are discovered from the filesystem, not configured (`config/workspace.py:137-143`): the workspace itself,
always, named by the root directory's basename; `product` when `<root>/product` is a directory; `harness` when
`<root>/context/harness/.git` exists.

## Domain

```rust
// product:configuration-validation, engineering:typed-outcomes
// Every configuration failure. A variant carries the values its message names; Display renders them.
// Nothing constructs a ConfigError from a pre-rendered string.
#[derive(Debug, thiserror::Error)]
pub enum ConfigError {
    RootNotFound { start: PathBuf },
    FileRead { path: PathBuf, cause: FileCause },                 // an existing file that cannot be read or parsed
    RemovedKey(RemovedKey),
    InvalidChoice { key: ChoiceKey, value: ConfigValue },         // adopt_extensions, tui.dashboard.layout
    EnvsPerWorkspaceTooSmall { envs_per_workspace: i64, aliases: usize },
    DuplicateStandaloneName { name: String, first: EntryIdent, second: EntryIdent },
    PrefixCollision { prefix: String, standalone: String },
    Repository { kind: RepoKind, label: String, fault: RepositoryFault },
    ValueType { key: TypedKey, owner: Option<RepoRef>, path: Option<PathBuf>, found: ValueType },  // see Value types
    EnvVar { band: String, key: String, fault: EnvVarFault },
    ModelTier { label: String, fault: ModelTierFault },
    AgentOverridesNotTable { path: PathBuf, found: ValueType },
    AgentOverride { agent: String, fault: AgentOverrideFault },
    ProvisionEntry(ProvisionEntryError),    // variants: extension-manifest.md, Provision entry
    ServiceEntry(ServiceEntryError),        // variants: extension-manifest.md, Service entry
    Lock(LockError),                        // variants: lock-and-state.md
}

// Why a TOML file could not be loaded. Defined here once; lock-and-state.md and extension-manifest.md use it.
// The message of either cause is the operating system's or the TOML library's own.
#[derive(Debug, thiserror::Error)]
pub enum FileCause {
    #[error(transparent)] Io(#[from] std::io::Error),             // exists but cannot be read
    #[error(transparent)] Toml(#[from] TomlParseError),           // read but not valid TOML
}
#[derive(Debug, thiserror::Error)]
#[error(transparent)]
pub struct TomlParseError(/* the TOML library's parse error, with its position */);

pub enum RemovedKey { ServiceOrchestrator, SessionPrefix, EnvVars { keys: Vec<String> } }
pub enum ChoiceKey  { AdoptExtensions, DashboardLayout }
pub enum RepoKind   { Project, Standalone }
pub enum EntryIdent { Name(String), Url(String) }
pub struct RepoRef  { pub kind: RepoKind, pub label: String }
pub enum TypedKey   { GitExcludes, Cmd, MainBranch, Ref, Name, Url, Prefix, Path, ConfigDir,
                      ProjectRepository, StandaloneRepository, Git, GitUser, GitUserName, GitUserEmail,
                      Keybindings, Tui, Capabilities, Env, ModelTiers }

pub enum RepositoryFault {
    Path { field: PathField, value: String },                     // path, config_dir: absolute or with a `..` segment
    StandaloneNested,
    NestedOnly { key: NestedOnlyKey },                            // envs, inherit_local without nested = true
    NestedWith { key: NestedExcludedKey },                        // load or entry on a nested = true repo
    NestedWithExtensionTrue,
    Extension { value: ConfigValue },
    Nested { value: ConfigValue },
    Load { value: ConfigValue },
    EntryList { value: ConfigValue },
    EntryPathEmpty { value: ConfigValue },
    EntryPathEscapes { value: String },
    Envs { value: ConfigValue },
    InheritLocal { value: ConfigValue },
}

pub enum EnvVarFault {
    UnsupportedType { found: ValueType },
    UnknownFields { fields: Vec<String> },
    ManagedName,                                                  // command entry under a WINTER_* key
    MissingCommand,
    InvalidFormat { value: ConfigValue },
    ShellNotBoolean { found: ValueType },
    ExportsNotStringList,
}

pub enum ModelTierFault {
    NotVendorTable { found: ValueType },
    UnknownVendor { vendor: String },
    ModelIdNotString { vendor: String, found: ValueType },
    NoVendor,
}

pub enum AgentOverrideFault {
    UnknownTier { label: String, known: Vec<String> },
    EmptyValue,
    WrongType { found: ValueType },
    UnknownVendor { vendor: String },
    NoVendor,
    VendorModelId { vendor: String },
    VendorValueType { vendor: String, found: ValueType },
    UnknownProfileKeys { vendor: String, keys: Vec<String> },
    ProfileModel { vendor: String },
    ProfileEffort { vendor: String },
    ProfileEmpty { vendor: String },
}

pub struct ConfigValue(/* a parsed TOML value */);
pub enum ValueType { Boolean, Integer, Float, String, Array, Table, Datetime }      // TOML's own type names
```

## Flows

### Root discovery

```text
# product:identify-workspace, decision:parallel-workspaces/workspace-configuration/root-selection
# reference: config/internal/cwd_workspace_locator.py:26-34
start = process working directory
for dir in [start, parent(start), …, "/"]:
    if <dir>/.winter/config.toml is a FILE: root = dir; stop        # a bare .winter/ directory does not qualify
none found -> ConfigError::RootNotFound, naming <start>             exit 1
```

The nearest qualifying ancestor wins, so an invocation inside a nested workspace resolves the nested root. There is no
root option and no root environment variable. Discovery runs once per invocation, on first need.

### Load

```text
# reference: config/workspace.py:125-330,380-400,438-452; core/internal/tomllib_config_file_reader.py:18-23
shared = parse(.winter/config.toml)          missing -> {}
local  = parse(.winter/config.local.toml)    missing -> {}
a file that exists and cannot be read  -> ConfigError::FileRead { cause: Io }
a file that is not valid TOML          -> ConfigError::FileRead { cause: Toml }
[agent_model_overrides] present and not a table, in either file -> ConfigError::AgentOverridesNotTable, naming the file
merged = merge(shared, local)                # Layering
then the typed configuration is built in this order, which decides the refusal reported when several apply:
  1. project_repository, its entries, then standalone_repository, its entries
                                                                        # Repository entries; config/workspace.py:146,170
  2. [git], git.user, git identity, keybindings, then tui.dashboard.layout
                                                                        # config/workspace.py:209-215; config/models.py:184-185
  3. capabilities
  4. removed keys: service_orchestrator, then session_prefix            # Removed keys
  5. adopt_extensions
  6. envs_per_workspace against env_aliases
  7. the workspace prefix against each standalone prefix
  8. core_checks, then the env bands, then space, model_tiers, agent_model_overrides
                                                                        # the removed [env.vars] is refused with the env bands
  9. the top-level main_branch                                          # Value types; config/workspace.py:323-330
 10. the top-level git_excludes                                         # Value types; config/workspace.py:327
a refusal is a ConfigError unless the key's table says otherwise        # Keys
[provision] and [[service]] stay raw; each is validated only when a command needs it
```

The first refusal met ends the load; one invocation reports one configuration error. Loading runs no external command
and writes nothing (`engineering:inert-configuration-read`). One invocation loads once and uses that value throughout
(`engineering:configuration-snapshot`); the lint ignore rules re-read and merge the same two files themselves
(`modules/lint/ignore_service.py:420-437`).

A key's value is kept in the loaded configuration only once a command reads it, and that command's spec names the key.
Until then the load applies the key's refusals and keeps nothing; a key whose rows hold no refusal is not read at all.
Neither changes what any invocation observes.

### Layering

Rules, per top-level key (`util.py:32-44`, `config/overlay.py:147-199`):

| Rule                   | Keys                                                                                              | Semantics                                                                                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Array append           | `project_repository`, `standalone_repository`                                                     | shared entries, then local entries. A local value that is not an array replaces the shared array.                                                           |
| Table merge, one level | `git`, `keybindings`, `tui`, `capabilities`, `env`, `model_tiers`, `agent_model_overrides`        | Local direct children replace shared direct children of the same name; others survive. A local value that is not a table leaves the shared table untouched. |
| Replace                | every other key, including `provision`, `service`, `lint`, `space`, `core_checks`, `git_excludes` | The local value replaces the shared value whole, whatever its type.                                                                                         |

An empty local file returns the shared document unchanged. One level means a nested table is replaced, not merged:

| Local file declares            | Effect on the shared value                                                         |
| ------------------------------ | ---------------------------------------------------------------------------------- |
| `[git] user = { name = "…" }`  | The shared `user.email` is dropped, so no identity exists.                         |
| `[keybindings.bindings]`       | Replaces the whole shared bindings table; `leader` and `timeoutlen` survive.       |
| `[env.feature.vars]`           | Replaces every shared feature var; `[env.workspace.vars]` and named bands survive. |
| `[capabilities] service = …`   | Replaces the shared `service` list; other slots survive.                           |
| `[space]`                      | Drops the shared `[space.kinds]`.                                                  |
| `lint = …`, `git_excludes = …` | Replace the shared value.                                                          |

```text
# config/overlay.py:170-183: the local file's keys are merged in its document order, and the first that fails is
# reported. For an array-append or table-merge key the local file holds, whatever the local value's type, the shared
# value is first read as that rule's kind; an empty shared value (false, zero, "", [], {}) reads empty.
# agent_model_overrides never fails here: a non-table in either file is refused before the merge (Load).
array-append key   shared true, a non-zero number, or a datetime -> ConfigError::ValueType naming the key and the
                   shared file; a shared string or table reads as its characters or keys, which step 1 skips
table-merge key    shared non-table -> ConfigError::ValueType naming the key and the shared file, except an array
                   whose every element is a pair: a two-character string, a two-key table, or a two-element array
                   whose first item is not an array or table; it reads as the table of those pairs, first item as key
```

## Data

### Top-level keys

"Falls back" means the default is used with no message.

| Key                  | Type                              | Default                                                    | Validation                                                                                                | On invalid                                          |
| -------------------- | --------------------------------- | ---------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | --------------------------------------------------- |
| `service_prefix`     | non-empty string                  | `"winter"`                                                 | —                                                                                                         | falls back                                          |
| `main_branch`        | string                            | `"main"`                                                   | an empty string, `false`, `0`, or an empty array or table is the default                                  | any other non-string → ConfigError; see Value types |
| `git_excludes`       | array of string                   | `[]`                                                       | an array whose every element is a string                                                                  | ConfigError; see Value types                        |
| `base_port`          | integer, not boolean              | `4000`                                                     | —                                                                                                         | falls back                                          |
| `ports_per_env`      | integer, not boolean              | `20`                                                       | —                                                                                                         | falls back                                          |
| `env_aliases`        | string or array of string         | `alpha beta gamma delta epsilon zeta eta theta iota kappa` | a bare string becomes a one-element list; non-string and empty items are dropped; other types become `[]` | coerced                                             |
| `envs_per_workspace` | integer, not boolean              | `48`                                                       | `>= len(env_aliases) + 2`                                                                                 | wrong type falls back; range → ConfigError          |
| `adopt_extensions`   | `none` \| `winter` \| `all`       | `winter`                                                   | enum                                                                                                      | ConfigError                                         |
| `prefix`             | non-empty string                  | `"ws"`                                                     | differs from every standalone `prefix`                                                                    | wrong type falls back; collision → ConfigError      |
| `skills_dir`         | non-empty string                  | `"skills"`                                                 | —                                                                                                         | falls back                                          |
| `doctor`             | string, workspace-relative path   | unset                                                      | path checked when the probe runs                                                                          | falls back to unset                                 |
| `lint`               | string, array of string, or table | `[]`                                                       | scalar or array = script paths; table reads `lint.scripts`                                                | coerced                                             |

### Tables

| Key path                                   | Type                                                      | Default     | Validation and on-invalid                                                                                                                                                     |
| ------------------------------------------ | --------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `git.user.name`, `git.user.email`          | string                                                    | unset       | Not a table at `[git]` or `git.user`, unless an empty form, or both non-empty and one not a string → ConfigError; see Value types. Else an identity when both are non-empty.  |
| `capabilities.<slot>`                      | string or array of string                                 | `{}`        | Normalised to an ordered, de-duplicated list; an empty result drops the slot; slot names unchecked at load. Known slot: `service`.                                            |
| `keybindings.leader`                       | string                                                    | `"\\"`      | Falls back.                                                                                                                                                                   |
| `keybindings.timeoutlen`                   | integer, not boolean                                      | `1000`      | Falls back.                                                                                                                                                                   |
| `keybindings.bindings.<action-id>`         | string (key spec)                                         | `{}`        | Keys and values coerced to strings.                                                                                                                                           |
| `tui.dashboard.layout`                     | `auto` \| `repos-as-columns` \| `repos-as-rows` \| `list` | `auto`      | ConfigError.                                                                                                                                                                  |
| `core_checks.file_size.injected_bytes`     | integer, not boolean                                      | `6000`      | Falls back.                                                                                                                                                                   |
| `core_checks.file_size.reference_bytes`    | integer, not boolean                                      | `12000`     | Falls back.                                                                                                                                                                   |
| `env.workspace.vars.<KEY>`                 | string, integer, float, or command-entry table            | `{}`        | Boolean, array, or other → ConfigError. Numbers become their decimal string.                                                                                                  |
| `env.feature.vars.<KEY>`                   | same                                                      | `{}`        | Same.                                                                                                                                                                         |
| `env.<name>.vars.<KEY>`                    | same                                                      | `{}`        | Any `[env]` sub-table other than `workspace` and `feature` is a per-env band; the name is not validated.                                                                      |
| `space.root`                               | non-empty string                                          | `".winter"` | Workspace-relative, `~`-relative, or absolute. Falls back.                                                                                                                    |
| `space.kinds.<kind>`                       | non-empty string                                          | `{}`        | Relative to the resolved root unless `~`-relative or absolute; non-string dropped.                                                                                            |
| `model_tiers.<label>.<vendor>`             | non-empty string                                          | `{}`        | Vendor is `claude`, `codex`, or `opencode`; at least one vendor per label; else ConfigError.                                                                                  |
| `agent_model_overrides.<agent>`            | tier label, or table `<vendor> = id \| { model, effort }` | `{}`        | A bare string must be a label of the effective tier table; a vendor table is non-empty; profile keys are closed to `model`, `effort` with at least one set; else ConfigError. |
| `lint.scripts`                             | string or array of string                                 | `[]`        | Coerced as the scalar `lint`.                                                                                                                                                 |
| `lint.ignore.paths`                        | array of glob                                             | unset       | Invalid → a lint finding (source `core`, check `lint-ignore`), not a ConfigError.                                                                                             |
| `lint.ignore.checks`                       | table: check → array of glob                              | unset       | Same.                                                                                                                                                                         |
| `lint.ignore.repos`                        | array of string                                           | unset       | Same.                                                                                                                                                                         |
| `lint.ignore.repo.<name>.paths`, `.checks` | as above, per repo                                        | unset       | Same.                                                                                                                                                                         |
| `provision.<subtarget>[]`                  | array of table                                            | `{}`        | Kept raw at load; a `[provision]` that is not a table reads as empty. Validated when `provision`, `clean`, or `doctor` needs it.                                              |
| `service[]`                                | array of table                                            | `[]`        | Kept raw at load; validated when `service up` needs it.                                                                                                                       |

Env command-entry table, under any `env.*.vars.<KEY>` (`config/workspace.py:425-560`):

| Field     | Type                        | Default | Rule                                             |
| --------- | --------------------------- | ------- | ------------------------------------------------ |
| `command` | non-empty string            | —       | Required. The key must not start with `WINTER_`. |
| `format`  | `raw` \| `dotenv` \| `json` | `raw`   |                                                  |
| `shell`   | boolean                     | `false` |                                                  |
| `exports` | array of string             | unset   |                                                  |

The field set is closed: any other field is a ConfigError. Band resolution, token grammar, and command-entry execution
belong to `epic:running-environments/scope-environment`.

Built-in model tiers, overlaid per vendor cell by `[model_tiers]`:

| Label    | `claude` | `codex`         | `opencode`                    |
| -------- | -------- | --------------- | ----------------------------- |
| `fable`  | `fable`  | `gpt-6-astra`   | `anthropic/claude-fable-5-1`  |
| `opus`   | `opus`   | `gpt-6.1-sol`   | `anthropic/claude-opus-5-5`   |
| `sonnet` | `sonnet` | `gpt-6.1-terra` | `anthropic/claude-sonnet-5-5` |
| `haiku`  | `haiku`  | `gpt-6-luna`    | `anthropic/claude-haiku-4-5`  |

### Repository entries

An element of either repository array that is not a table is skipped silently, as is an entry whose `name` and `url` are
both empty (a form Python reads as false; config/workspace.py:151,175). The resolved name is `name` when not empty, else
derived from `url` by step 2 of the effective-name rule of
[repository-removal.md](../../../../work/parallel-workspaces/repository-catalog/spec/repository-removal.md)
(config/workspace.py:177,773-781).

| Field           | `[[project_repository]]` | `[[standalone_repository]]` | Type, default                      | Rule                                                                                                                |
| --------------- | ------------------------ | --------------------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------------------- |
| `name`          | yes                      | yes                         | string                             | Standalone: the resolved name is unique, else ConfigError. Project: a duplicate name is **accepted**.               |
| `url`           | yes                      | yes                         | string                             |                                                                                                                     |
| `main_branch`   | yes                      | yes                         | string, workspace `main_branch`    |                                                                                                                     |
| `git_excludes`  | yes                      | yes                         | array of string, `[]`              |                                                                                                                     |
| `cmd`           | yes                      | yes                         | array of string, `[]`              | Each entry runs through the shell in the checkout or worktree.                                                      |
| `extension`     | yes                      | yes                         | boolean, `true`                    | Not a boolean → ConfigError.                                                                                        |
| `load`          | yes                      | yes                         | `eager` \| `lazy` \| `none`, unset | Else ConfigError.                                                                                                   |
| `entry`         | yes                      | yes                         | non-empty array of string, unset   | Each a non-empty repo-relative path, not absolute, no `..` segment; else ConfigError.                               |
| `pinned`        | yes                      | —                           | boolean, `false`                   | Coerced by truthiness.                                                                                              |
| `nested`        | yes                      | rejected when `true`        | boolean, `false`                   | Not a boolean → ConfigError. `true` forces `extension = false` and forbids `load`, `entry`, and `extension = true`. |
| `envs`          | yes                      | rejected                    | integer ≥ 1, not boolean, unset    | Only with `nested = true`.                                                                                          |
| `inherit_local` | yes                      | rejected                    | array of string, unset             | Only with `nested = true`; each name non-empty and without `.`. Unset means `["git"]` when delegating.              |
| `ref`           | —                        | yes                         | string, unset                      | Branch, tag, or commit; resolved into `config.lock`.                                                                |
| `path`          | —                        | yes                         | string, `<name>`                   | Relative, no `..` segment; else ConfigError.                                                                        |
| `prefix`        | —                        | yes                         | string, unset                      | Differs from the workspace `prefix`; else ConfigError.                                                              |
| `config_dir`    | —                        | yes                         | string, `.winter/config/<name>`    | Relative, no `..` segment; else ConfigError.                                                                        |

A value of the wrong TOML type for `name`, `url`, `main_branch`, `git_excludes`, `cmd`, `ref`, `path`, `prefix`, or
`config_dir` is a ConfigError; see Value types. Catalog semantics beyond these field rules belong to
`epic:parallel-workspaces/repository-catalog`.

### Value types

```text
# ConfigError::ValueType { key, owner, path, found }: names the key, the repository for a repository field, and the
# shared file for a refusal at the merge (Layering). Exit 1.
# "empty" below is a value Python reads as false: `false`, zero, "", or an empty array or table
key                                            accepted                 reference tool, for anything else
top-level git_excludes                         an array of strings      a string is read as one entry per character and
<repo>.git_excludes                            an array of strings        a table as its keys; `false`, `0`, "" read as
<repo>.cmd                                     an array of strings        empty; another type, or a non-string element,
                                                                          fails unhandled, exit 1
<repo>.main_branch, <standalone>.ref           a string                 fails unhandled, exit 1
top-level main_branch                          a string                 fails unhandled, exit 1, unless the value is
                                                                          one of the empty forms the key table lists
<repo>.name, .url, <standalone>.prefix,        a string                 fails unhandled, exit 1
  .path, .config_dir
project_repository, standalone_repository      an array, or a value     `true`, a non-zero number, or a datetime fails
                                                 the reference reads      unhandled, exit 1; any other value is iterated,
                                                 as no entries            and its items, never tables, are all skipped
                                                                          (config/workspace.py:146,170)
[git], git.user                                a table                  fails unhandled, exit 1, unless the value is
                                                                          empty, which reads as no identity
                                                                          (config/workspace.py:209-211)
git.user.name, git.user.email                  a string, or any value   fails unhandled, exit 1, when both are non-empty
                                                 while either is empty    and either is not a string; with either empty
                                                                          there is no identity (config/models.py:184-185)
```

```text
# decision:parallel-workspaces/workspace-configuration/element-type-validation      (registered deviation)
For git_excludes and cmd the binary refuses every value that is not an array of strings, including the string,
table, and empty non-array forms the reference tool accepts. An absent key and an empty array are not errors.
```

Every other row has the same outcome in both tools, a failed load with exit 1, and differs only in the diagnostic. Keys
with their own coercion rule in the tables above (`env_aliases`, `capabilities.<slot>`, `lint`, and the keys that fall
back) keep that rule.

### Unknown keys

```text
# decision:parallel-workspaces/workspace-configuration/validation-boundary
unknown top-level key, unknown key in any open table        -> ignored, no message
closed sets, unknown key -> ConfigError:                    env command entry; agent override profile;
                                                            provision entry; service entry
closed sets, unknown key -> lint finding, not an error:     [lint], [lint.ignore], [lint.ignore.repo.<name>]
```

A provision or service entry fault is a `ConfigError` value. In workspace configuration, a malformed `[provision]` entry
and a malformed `[[service]]` entry each reach the caller as a command error, exit 1, that carries the fault
(`modules/provision/provision_service.py:494-497`); in an extension manifest they are a manifest failure
([extension manifest](../../../../work/agent-ready-operation/extension-management/spec/extension-manifest.md), Failure
handling).

### Removed keys

Checked on the merged document, so a key present only in the local file is rejected too
(`config/workspace.py:240-249,445-452`). Each is a `ConfigError::RemovedKey` whose message names the replacement.

| Key                    | Replacement to name                                                        |
| ---------------------- | -------------------------------------------------------------------------- |
| `service_orchestrator` | `[capabilities] service = …`                                               |
| `session_prefix`       | `service_prefix`                                                           |
| `[env.vars]`           | `[env.feature.vars]` and `[env.workspace.vars]`; the keys found are listed |

### Cross-key rules

Conditions not stated in a key row, each a ConfigError (`config/workspace.py`):

| Variant                                  | Condition                                                                                   |
| ---------------------------------------- | ------------------------------------------------------------------------------------------- |
| `EnvsPerWorkspaceTooSmall`               | `envs_per_workspace < len(env_aliases) + 2`                                                 |
| `DuplicateStandaloneName`                | two standalone entries resolve to the same name, by `name` or by URL                        |
| `PrefixCollision`                        | the workspace `prefix` equals a standalone's `prefix`                                       |
| `Repository` / `StandaloneNested`        | a standalone entry with `nested = true`                                                     |
| `Repository` / `NestedOnly`              | `envs` or `inherit_local` on an entry whose `nested` is absent or `false`                   |
| `Repository` / `NestedWith`              | `load` or `entry` on a `nested = true` entry                                                |
| `Repository` / `NestedWithExtensionTrue` | `extension = true` on a `nested = true` entry                                               |
| `EnvVar` / `ManagedName`                 | a command-entry table under a key that starts with `WINTER_`                                |
| `AgentOverride` / `UnknownTier`          | a bare string that is not a label of the effective tier table; the known labels are carried |

### Derived values

Each is built with the first command that reads it: `port_base` by `epic:running-environments/scope-environment`,
`space_dir` by `epic:agent-ready-operation/artifact-space`, and `config_dir` by
`epic:agent-ready-operation/executable-extensions`.

```text
port_base(index)  = base_port + index * ports_per_env                          # config/models.py:910
space_dir(kind)   = root' / (space.kinds[kind] or kind)                        # config/models.py:917
                    root' = space.root, resolved against the workspace root unless ~-relative or absolute
                    a kind override that is ~-relative or absolute is used as is
config_dir(ext)   = <workspace root> / (config_dir or ".winter/config/<name>") # modules/workspace/repository_factory.py:115
```

## Interfaces

### Writers of `config.toml` and `config.local.toml`

```text
# product:configuration-state-ownership, engineering:configuration-write-integrity
# decision:parallel-workspaces/workspace-configuration/write-conflicts
every writer parses the existing file, edits the document, and serialises it back:
  comments, blank lines, key order, quoting style, and unrelated tables are preserved byte for byte
```

| Operation                             | Target                          | Edit                                                                                                                                                                                                  | Reference                                                        |
| ------------------------------------- | ------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Append a repository                   | shared, or local with `--local` | Append one `[[project_repository]]` or `[[standalone_repository]]` block at the end, holding only fields that differ from their defaults and are set.                                                 | `config/internal/write_winter_configuration_repository.py:40-70` |
| Remove a repository                   | shared, or local with `--local` | Remove the block whose `name`, or URL-derived name, matches.                                                                                                                                          | same                                                             |
| Set a standalone `ref`                | the file declaring the entry    | Insert `ref` after the block's last key.                                                                                                                                                              | same; `modules/workspace/workspace_sync_service.py:671-672`      |
| Upsert local keys (nested delegation) | a nested workspace's local file | A same-form value is edited in place; otherwise the key is deleted and re-inserted; a new scalar goes before the first table, a new table at the end. No write when the serialised text is unchanged. | `config/internal/tomlkit_local_overlay_repository.py:42-118`     |

The local file and its directory are created when absent. The shared file must exist; a missing shared file is an
internal failure, not a created file. Nested delegation writes `base_port` and `service_prefix`, plus `env_aliases = []`
and `envs_per_workspace = envs + 1` when `envs` is set, plus the inherited outer-local tables (default `git`; never the
two repository arrays) (`modules/workspace/nested_overlay.py:17-59`); its policy belongs to
`epic:parallel-workspaces/nested-workspaces`.

Each writer is built with the first command that calls it: appending a repository with `repo add`, by
`epic:parallel-workspaces/repository-catalog`; setting a standalone `ref` with `ws update`, by
`epic:parallel-workspaces/reproducible-inputs`; and the upsert with nested delegation, by
`epic:parallel-workspaces/nested-workspaces`. Removing a repository is built, with `repo remove`.

### Write integrity

```text
# engineering:concurrent-state-safety, decision:parallel-workspaces/workspace-configuration/write-conflicts
# applies to all four files
lock:     an exclusive advisory lock (flock) on an open handle of the directory that holds the target, `.winter/`
          the directory is created first when it is absent (local file, lock, state), so the lock never depends
          on the target existing; no lock file is created; the lock ends when the handle closes or the process dies
          held across the whole read-modify-write of one file; one lock serialises writers of all four files
          a writer that cannot take the lock waits; it never fails and never writes a message
write:    serialise to a temporary file in the directory of the resolved target, flush, rename over the resolved target
          the temporary file never survives a completed or failed write; the target keeps its permission bits;
          a target that did not exist is created with the mode a plain create would give it
symlink:  a target that is a symbolic link is resolved first; the temporary file is written beside the file the link
          points to and renamed over that file; the link itself is left in place
readers:  take no lock; a reader sees the old file or the new file, never a partial one
visible:  file content, file set, and exit codes are identical to an unlocked, non-atomic writer
          when there is no concurrent writer and no interruption
```

Write integrity is built in the file port, with `repo remove`, and each writer of the four files uses it as it lands.

## Open questions

- The placement of a key inserted by the upsert writer into a document that already mixes inline and standard tables is
  described only by the rule above. Engineering pins the remaining cases with recorded scenarios when
  `epic:parallel-workspaces/nested-workspaces` is specified.
- `decision:agent-ready-operation/workspace-dashboard/contribution-scope` says an ignored plugin produces one notice,
  but not what triggers it, which stream carries it, or its text. The product owner answers when
  `epic:agent-ready-operation/workspace-dashboard` is refined; until then the binary writes nothing for a plugin path.
