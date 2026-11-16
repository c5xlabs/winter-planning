# Configuration model

The workspace configuration files, their keys, layering, validation, and writers. The binary reads and writes the
reference tool's files unchanged (`decision:configuration-transition`). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`. The lock and machine-state files
are [lock-and-state.md](./lock-and-state.md); error rendering is
[results-and-exit-codes.md](../../component-runtime/spec/results-and-exit-codes.md).

Under `decision:functional-parity` the contract is which values are accepted, coerced, defaulted, or refused, the error
kind of each refusal with the values it carries, and the bytes written to the files. The wording of a configuration
error is the binary's own; the reference texts are in `config/workspace.py`.

The modules that locate, read, merge, and build the configuration, their seams, and the commands that exercise them are
[discovery-and-loading.md](./discovery-and-loading.md); the replacement of a file is [write-path.md](./write-path.md).

## System

| File                        | Format | Holds                            | Committed | Written by                                                                                                             |
| --------------------------- | ------ | -------------------------------- | --------- | ---------------------------------------------------------------------------------------------------------------------- |
| `.winter/config.toml`       | TOML   | Shared workspace declaration     | yes       | `repo add`, `repo remove`; `ws update --freeze` for a standalone's `ref`                                               |
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
    FileWrite { path: PathBuf, cause: std::io::Error },           // a file that cannot be replaced; write-path.md
    RemovedKey(RemovedKey),
    InvalidChoice { key: ChoiceKey, value: ConfigValue },         // adopt_extensions, tui.dashboard.layout
    EnvsPerWorkspaceTooSmall { envs_per_workspace: i64, aliases: usize },
    DuplicateStandaloneName { name: String, first: EntryIdent, second: EntryIdent },
    PrefixCollision { prefix: String, standalone: String },
    RepositoriesNotArray { kind: RepoKind, file: ConfigFileName, found: ValueType },   // see Layering
    Repository { kind: RepoKind, label: String, fault: RepositoryFault },
    ValueType { key: TypedKey, owner: Option<RepoRef>, found: ValueType },   // see Value types
    EnvVar { band: String, key: String, fault: EnvVarFault },
    ModelTier { label: String, fault: ModelTierFault },
    AgentOverridesNotTable { file: ConfigFileName, found: ValueType },
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
                      Table(MergedTable), GitUser, GitUserName, GitUserEmail }
pub enum MergedTable { Git, Keybindings, Tui, Capabilities, Env, ModelTiers, AgentModelOverrides }
pub enum ConfigFileName { Shared, Local }

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

pub struct ConfigValue { pub text: String, pub found: ValueType }   // the value as TOML text; no library type
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
none found -> ConfigError::RootNotFound; its message names <start> and the file looked for,
              .winter/config.toml, as the reference tool's does     exit 1
```

The nearest qualifying ancestor wins, so an invocation inside a nested workspace resolves the nested root. There is no
root option and no root environment variable. Discovery runs once per invocation, on first need.

### Load

```text
# reference: config/workspace.py:125-321,380-400,438-452,768; core/internal/tomllib_config_file_reader.py:18-23
shared = parse(.winter/config.toml)          missing, or not a regular file -> {}
local  = parse(.winter/config.local.toml)    missing, or not a regular file -> {}
                           # decision:parallel-workspaces/workspace-configuration/non-regular-configuration-path
a regular file that cannot be read             -> ConfigError::FileRead { cause: Io }
a file that is not UTF-8, or not valid TOML    -> ConfigError::FileRead { cause: Toml }
[agent_model_overrides] present and not a table, in either file -> ConfigError::AgentOverridesNotTable, naming the file
project_repository or standalone_repository present and not an array, in either file
                                               -> ConfigError::RepositoriesNotArray, naming the key and the file
merged = merge(shared, local)                # Layering
then the typed configuration is built in this order, which decides the refusal reported when several apply:
  1. project repository entries, then standalone repository entries     # Repository entries
  2. git identity, keybindings, then tui.dashboard.layout               # an invalid layout is refused here
  3. capabilities
  4. removed keys: service_orchestrator, then session_prefix            # Removed keys
  5. adopt_extensions
  6. envs_per_workspace against env_aliases
  7. the workspace prefix against each standalone prefix
  8. core_checks, then the env bands, then space, model_tiers, agent_model_overrides
                                                                        # the removed [env.vars] is refused with the env bands
  9. the types of the top-level git_excludes, then main_branch          # Value types
a refusal is a ConfigError unless the key's table says otherwise        # Keys
[provision] and [[service]] stay raw; each is validated only when a command needs it
```

The first refusal met ends the load; one invocation reports one configuration error. Loading runs no external command
and writes nothing (`engineering:inert-configuration-read`). One invocation loads once and uses that value throughout
(`engineering:configuration-snapshot`); the lint ignore rules re-read and merge the same two files themselves
(`modules/lint/ignore_service.py:420-437`).

Which refusal is reported when several apply cannot be told from the ending or the exit code, which are the same for all
of them; the order above is the binary's own and is fixed so that one configuration always gives one message.

### Layering

Rules, per top-level key (`util.py:32-44`, `config/overlay.py:147-199`):

| Rule                   | Keys                                                                                              | Semantics                                                                                                                                                   |
| ---------------------- | ------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Array append           | `project_repository`, `standalone_repository`                                                     | shared entries, then local entries. A value that is not an array is a ConfigError in either file.                                                           |
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

A key under a merge rule whose shared value has the wrong type (`config/overlay.py:165-187`). "Empty form" is `false`,
`0`, `0.0`, `""`, `[]`, or `{}`; the same forms are what "empty" means for the top-level `main_branch` and for `pinned`.
They are not what "absent" means for a string field of a repository entry (Repository entries).

```text
table-merge key, the local file holds the key at all:
    shared value absent or an empty form        -> merged as if the shared value were {}
    shared value any other non-table            -> ConfigError::ValueType { key: Table(<key>) }
                                                   # decision:…/unhandled-reference-failures
        a shared array of pairs among them      # decision:…/element-type-validation    (registered deviation)
    the local file does not hold the key        -> the shared value passes to the load unchanged
array-append key, in either file:               # decision:…/repository-array-type      (registered deviation)
    absent, or an array                         -> appended: shared entries, then local entries
    any other value, the empty forms included   -> ConfigError::RepositoriesNotArray, naming the key and the file;
                                                   checked in each file before the merge, the shared file first,
                                                   and in each file project_repository first
```

The reference tool lets a local value that is not an array replace the shared array, reads a string or a table as no
repositories, and fails unhandled on a number other than zero. For a table-merge key the local file holds, it builds a
table from the shared value (`config/overlay.py:177`): a shared array whose every element is a pair, such as
`keybindings = [["leader", "x"]]`, becomes the table of those pairs and the load goes on, where every other shared
non-table that is not an empty form fails unhandled. The binary refuses the array of pairs with the rest. This reaches
six of the seven keys: a shared `agent_model_overrides` that is not a table is refused by the check made in each file,
in both tools.

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

`base_port`, `ports_per_env`, and `envs_per_workspace` have no range check beyond the one in the table: zero and
negative values are accepted. An empty `env_aliases` string is an empty list. An alias that appears twice is kept twice;
the index it resolves to is in [lock-and-state.md](./lock-and-state.md).

### Tables

| Key path                                   | Type                                                      | Default     | Validation and on-invalid                                                                                                                                                     |
| ------------------------------------------ | --------------------------------------------------------- | ----------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `git.user.name`, `git.user.email`          | string                                                    | unset       | An identity exists only when both are non-empty; otherwise none, silently.                                                                                                    |
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

A table key whose value is not a table, as the build receives it after the merge:

```text
# reference: config/workspace.py:209,223,294-300,368-371,387-391,410-414,442,476-481,572-579,620
keybindings, tui, capabilities, core_checks, env, space, model_tiers
                                                -> read as absent: the defaults of the rows above, no message
tui.dashboard, keybindings.bindings, core_checks.file_size,
  env.<band>, env.<band>.vars, space.kinds      -> read as absent: the defaults, no message
git, git.user                                   -> an empty form reads as absent: no identity, no message;
                                                   any other non-table -> ConfigError::ValueType; see Value types
agent_model_overrides                           -> ConfigError::AgentOverridesNotTable, in the file that holds it
provision                                       -> read as empty
service that is not an array                    -> read as []
```

The first two rows are what the reference tool does, and no deviation. A non-table shared value of a table-merge key
reaches the build only when the local file does not hold the key; when it does, Layering decides.

Env command-entry table, under any `env.*.vars.<KEY>` (`config/workspace.py:425-560`):

| Field     | Type                        | Default | Rule                                             |
| --------- | --------------------------- | ------- | ------------------------------------------------ |
| `command` | non-empty string            | —       | Required. The key must not start with `WINTER_`. |
| `format`  | `raw` \| `dotenv` \| `json` | `raw`   |                                                  |
| `shell`   | boolean                     | `false` |                                                  |
| `exports` | array of string             | unset   |                                                  |

The field set is closed: any other field is a ConfigError. Band resolution, token grammar, and command-entry execution
belong to `epic:running-environments/scope-environment`.

The decimal string of a number under `env.*.vars` reaches child processes, so its form is function:

```text
integer        its decimal digits, with a leading `-` when negative
float          the shortest digits that read back as the same value, as the reference tool's str() gives them:
               an integral value ends in `.0`                          1.0 -> "1.0"      -0.0 -> "-0.0"
               exponent form when the decimal exponent is below -4 or at least 16, the exponent signed and
               at least two digits                                     1e16 -> "1e+16"   0.00001 -> "1e-05"
               inf -> "inf"      -inf -> "-inf"      nan -> "nan"
```

In `model_tiers` a model id made only of white space is accepted; under `agent_model_overrides` it is refused
(`VendorModelId`, `ProfileModel`).

Built-in model tiers, overlaid per vendor cell by `[model_tiers]`:

| Label    | `claude` | `codex`         | `opencode`                    |
| -------- | -------- | --------------- | ----------------------------- |
| `fable`  | `fable`  | `gpt-6-astra`   | `anthropic/claude-fable-5-1`  |
| `opus`   | `opus`   | `gpt-6.1-sol`   | `anthropic/claude-opus-5-5`   |
| `sonnet` | `sonnet` | `gpt-6.1-terra` | `anthropic/claude-sonnet-5-5` |
| `haiku`  | `haiku`  | `gpt-6-luna`    | `anthropic/claude-haiku-4-5`  |

### Repository entries

```text
# reference: config/workspace.py:146-207
for each element of project_repository, then of standalone_repository, in document order:
    the element is not a table                  -> skipped, no message
    name, then url, present and not a string    -> ConfigError::ValueType { key: Name | Url, owner }    # Value types
    name and url each absent or ""              -> skipped whole, no message; no other field of it is read
    every other field                           -> its row below
owner.label, and the label of a Repository fault:
    the name when it is a non-empty string, else the url when it is a non-empty string,
    else `#<n>`, the position of the element in its array, counted from one
```

`name` defaults to the URL's last segment: one trailing `/` run and then one trailing `.git` are removed, and the name
is what follows the last `/` or `:` that remains.

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
`config_dir` is a ConfigError, the empty forms included; see Value types. An empty string for `main_branch`, `ref`,
`path`, `prefix`, or `config_dir` is a string and is read as the field's row reads it. Catalog semantics beyond these
field rules belong to `epic:parallel-workspaces/repository-catalog`.

### Value types

```text
# ConfigError::ValueType { key, owner, found }: names the key and, for a repository field, the repository. Exit 1.
key                                            accepted                 reference tool, for anything else
top-level git_excludes                         an array of strings      a string is read as one entry per character and
<repo>.git_excludes                            an array of strings        a table as its keys; `false`, `0`, "" read as
<repo>.cmd                                     an array of strings        empty; another type, or a non-string element,
                                                                          fails unhandled, exit 1
top-level main_branch                          a string                 fails unhandled, exit 1, unless the value is
                                                                          one of the empty forms the key table lists
<repo>.name, .url                              a string                 an entry whose name and url are both empty
                                                                          forms is skipped whole, whatever their types
                                                                          (config/workspace.py:149-152,173-176); any
                                                                          other non-string is believed to fail the
                                                                          model's validation unhandled, exit 1
<repo>.main_branch, <standalone>.ref,          a string                 believed to fail the model's validation
  .prefix                                                                 unhandled, exit 1, the empty forms included
<standalone>.path, .config_dir                 a string                 fails unhandled, exit 1
                                                                          (config/workspace.py:188-193,889-891)
```

```text
# decision:parallel-workspaces/workspace-configuration/element-type-validation      (registered deviation)
For git_excludes and cmd the binary refuses every value that is not an array of strings, including the string,
table, and empty non-array forms the reference tool accepts. An absent key and an empty array are not errors.

For the string fields of a repository entry the binary refuses every value that is not a string, `false`, `0`,
`0.0`, `[]`, and `{}` included. Read in the reference source, one case passes there: an entry whose name and url
are both empty forms and one of them is not a string, such as `name = 0` with no url, is skipped; here it is refused.

"Believed" marks what rests on the validation rules of the reference tool's model library, which were not read in
its source (config/models.py:214-224,313-341 declares each field as a string or nothing). It is to be confirmed
by a probe: the recorded scenarios ws-index/repository-name-zero-with-url, a project entry with `name = 0` and a
url, and ws-index/repository-main-branch-zero, a project entry with `main_branch = 0`. A case a record shows the
reference tool accepting is a further case of this deviation, and the binary refuses it all the same.
```

Apart from those cases every row has the same outcome in both tools, a failed load with exit 1, and differs only in the
diagnostic. Keys with their own coercion rule in the tables above (`env_aliases`, `capabilities.<slot>`, `lint`, and the
keys that fall back) keep that rule.

```text
# decision:parallel-workspaces/workspace-configuration/unhandled-reference-failures
# a refusal is the same outcome in both tools, exit 1; the reference tool fails unhandled (config/workspace.py:179-212)
# an empty form of git or of git.user is no refusal in either tool: it reads as absent, so no identity, no message
#   (config/workspace.py:209); any other non-table, a string, number, boolean, array, or datetime, has no `get`
#   there, and the reference tool fails unhandled on git at line 209 and on git.user at line 211
key                                            accepted                 refused as
git                                            a table, an empty form,  ValueType { key: Table(Git) }
                                                 or absent
git.user                                       a table, an empty form,  ValueType { key: GitUser }
                                                 or absent
git.user.name, git.user.email                  a string                 ValueType { key: GitUserName | GitUserEmail },
                                                                          only when both values are not empty forms
```

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

```text
# decision:parallel-workspaces/workspace-configuration/write-byte-preservation      (registered deviation)
# reference: core/internal/local_filesystem.py:39-41,59-61; config/internal/write_winter_configuration_repository.py:68-112
a writer edits the bytes it reads under the lock of write-path.md, never the loaded configuration
every byte outside the edit is written back as it was read:
    line endings     a line that ends in CR LF keeps it; a file of mixed line endings keeps each
    encoding         the file is UTF-8, as the load requires, and is written as UTF-8 whatever the locale;
                     a byte order mark at the start is kept
reference tool       reads and writes the file as text: every line ending is written as the platform's, LF on
                     Linux, and the bytes are decoded and encoded in the encoding of the locale
a failed replacement -> ConfigError::FileWrite
```

## Open questions

- The placement of a key inserted by the upsert writer into a document that already mixes inline and standard tables is
  described only by the rule above. Engineering pins the remaining cases with recorded scenarios when
  `epic:parallel-workspaces/nested-workspaces` is specified.
- `decision:agent-ready-operation/workspace-dashboard/contribution-scope` says an ignored plugin produces one notice,
  but not what triggers it, which stream carries it, or its text. The product owner answers when
  `epic:agent-ready-operation/workspace-dashboard` is refined; until then the binary writes nothing for a plugin path.
- Which line ending the lines of a block take that a writer adds to a file with CR LF line endings is not stated: the
  removal, the one writer this epic builds, adds no line. Engineering states it when
  `epic:parallel-workspaces/repository-catalog` specifies the append writer.
