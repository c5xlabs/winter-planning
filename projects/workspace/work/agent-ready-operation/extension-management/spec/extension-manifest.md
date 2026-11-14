# Extension manifest

The `winter-ext.toml` file an extension repository declares itself with: every key, its default, its validation, and
what a failure does. The format is the reference tool's, unchanged
(`decision:agent-ready-operation/extension-management/manifest-policy`). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`; the manifest reader is
`modules/workspace/extension_manifest.py:292-400`. How the declared programs are run is the
[invocation contract](../../executable-extensions/spec/invocation-contract.md); the `service` slot is the
[provider contract](../../../running-environments/service-orchestration/spec/provider-contract.md).

The manifest is a contract toward extension authors: the keys, what is accepted, and what is refused are fixed. The
wording of a refusal is the binary's own (`decision:functional-parity`); the reference texts are in the files cited per
section.

## System

```text
location:   <checkout root>/winter-ext.toml
            checkout root = a standalone repository's clone directory, or a project repository's source checkout
candidates: every declared repository whose workspace entry does not set `extension = false`,
            filtered by the workspace key `adopt_extensions` (none | winter | all)
            # decision:agent-ready-operation/extension-management/adoption-policy
absent:     the repository contributes context by the default entry point only
```

Default entry point, when neither the workspace entry nor the manifest names one: the first that exists of `index.md`,
`AGENTS.md`, `context/index.md`.

## Domain

```rust
// engineering:typed-outcomes, engineering:manifest-validation
// Every way reading one winter-ext.toml fails. Carried as the typed cause of a RepoError;
// ExcludesDeclaration keeps its own kind so callers can match it.
#[derive(Debug, thiserror::Error)]
pub enum ManifestError {
    File(FileCause),                    // exists but cannot be read, or is not valid TOML
    Load { value: ConfigValue },
    Provision(ProvisionEntryError),
    Service(ServiceEntryError),
    ExcludesDeclaration(ExcludesFault),
}

pub enum ExcludesFault {
    NotTable { found: ValueType },
    UnknownFields { fields: Vec<String> },
    NotStringList { key: ExcludesKey, found: ValueType },
    PathEmpty { value: ConfigValue },
    Path { value: String, fault: ExcludesPathFault },
    SpaceKind { value: ConfigValue },
}
pub enum ExcludesPathFault { ControlCharacter, Negated, Comment, HomeDriveOrUnc, ParentSegment, RootOnly }

// `source` is "workspace" or the declaring extension's prefix; `index` counts from 0.
pub enum ProvisionEntryError {
    UnknownSubtarget { source: String, key: String },
    NotTableList { source: String, key: String, found: ValueType },
    NotTable { source: String, key: String, index: usize, found: ValueType },
    UnknownKeys { source: String, key: String, index: usize, keys: Vec<String> },
    MissingScope { source: String, key: String, index: usize },
    InvalidScope { source: String, key: String, index: usize, value: ConfigValue },
    Command { source: String, key: String, index: usize, field: CommandField, fault: CommandFault },
    RequiredServicesNotAllowed { source: String, key: String, index: usize },
    RequiredServicesType { source: String, key: String, index: usize },
    ProjectNotAllowed { source: String, key: String, index: usize, scope: String },
    ProjectType { source: String, key: String, index: usize },
    ProjectUnknown { source: String, key: String, index: usize, project: String, declared: Vec<String> },
    NameType { source: String, key: String, index: usize },
    NameDuplicate { source: String, key: String, index: usize, name: String, scope: String },
}
pub enum CommandField { Apply, Destroy, Reset, Clean }
pub enum CommandFault {
    Missing, EmptyString, EmptyList,
    ItemNotString { item: usize, found: ValueType },
    WrongType { found: ValueType },
}

pub enum ServiceEntryError {
    NotTableList { source: String, found: ValueType },
    NotTable { source: String, index: usize, found: ValueType },
    UnknownKeys { source: String, index: usize, keys: Vec<String> },
    MissingName { source: String, index: usize },
    InvalidScope { source: String, index: usize, value: ConfigValue },
    FieldNotString { source: String, index: usize, field: ServiceStringField, found: ValueType },
    PortsNotStringList { source: String, index: usize },
    DuplicateName { name: String, sources: Vec<String> },
}
```

`FileCause`, `ConfigValue`, and `ValueType` are defined in the
[configuration model](../../../parallel-workspaces/workspace-configuration/spec/configuration-model.md).

## Data

### Top-level keys

| Key                    | Type                              | Default                         | Rule                                                                                                           |
| ---------------------- | --------------------------------- | ------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| `name`                 | string                            | the repository's catalog name   | Read only as a step of Prefix resolution.                                                                      |
| `prefix`               | string                            | see Prefix resolution           |                                                                                                                |
| `description`          | string                            | unset                           |                                                                                                                |
| `load`                 | `eager` \| `lazy`                 | undeclared                      | Absent or `""` is undeclared. Any other value fails the manifest. `none` is valid only in the workspace entry. |
| `skills_dir`           | string                            | `skills`, then `.claude/skills` | When set, replaces both defaults. Not type-checked.                                                            |
| `agents_dir`           | string                            | `agents`, then `.claude/agents` | Same.                                                                                                          |
| `doctor`               | non-empty string, repo-relative   | unset                           | Checked when the probe runs.                                                                                   |
| `lint`                 | string, array of string, or table | `[]`                            | Scalar or array = script paths. Table reads `lint.scripts`, `lint.ignore.paths`, `lint.ignore.checks`.         |
| `requires`             | array of string                   | `[]`                            | Not an array → `[]`; non-string and empty items dropped.                                                       |
| `orchestrate_services` | non-empty string, repo-relative   | unset                           | Legacy alias of `provides.service`; read only when `provides.service` is absent.                               |

### Tables

| Table          | Keys                                                                       | Rule                                                                                                 |
| -------------- | -------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- |
| `[provides]`   | `<slot> = "<repo-relative entrypoint>"`                                    | One program per slot. The known slot is `service`. Non-string values are dropped.                    |
| `[implements]` | `<slot> = "<version>"`                                                     | Absent means compatible with every supported version. Non-string values are dropped.                 |
| `[hooks]`      | `on_env_init`, `on_env_destroy`, `on_workspace_reconcile`                  | Repo-relative script paths. Other string keys are kept and never run; non-string values are dropped. |
| `[provision]`  | `[[provision.dependency]]`, `[[provision.resource]]`, `[[provision.data]]` | Entry schema below. A malformed entry fails the whole manifest.                                      |
| `[[service]]`  | entry schema below                                                         | A malformed entry fails the whole manifest.                                                          |
| `[excludes]`   | `paths`, `space_kinds`                                                     | Closed. Rules below. A malformed table fails the whole manifest, as an excludes declaration failure. |

### Prefix resolution

```text
# reference: modules/workspace/extension_manifest.py:318; modules/workspace/repository_factory.py:112-120,309-336
first that is set: workspace entry `prefix`  >  manifest `prefix`  >  manifest `name`  >  catalog name
catalog name  = the repository entry's declared `name`, else the name derived from its `url`
                it is not the directory the repository is checked out in: a standalone entry with `path` set is
                checked out under <root>/<path> and still falls back to its catalog name
overlap(a, b) = a == b, or one starts with the other followed by "-"
```

The resolved prefix is also the `source` of the extension's provision and service entries, in their faults and in the
service manifest file.

### `[excludes]`

| Field         | Type            | Rule                                                                                                                                                                                                             |
| ------------- | --------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `paths`       | array of string | Each non-empty; no newline, carriage return, or NUL; no `!`; no leading `#`; no `~`, backslash, drive prefix, or `//`; no `..` segment; not the root alone. Normalised to a leading `/`; de-duplicated in order. |
| `space_kinds` | array of string | Each matches `^[A-Za-z0-9][A-Za-z0-9._-]*$`.                                                                                                                                                                     |

Each broken rule is one `ExcludesFault` (`modules/workspace/extension_manifest.py:34-290`): the table is not a table; it
holds a field other than the two; a field is not an array of strings; a path is empty or not a string; a path breaks one
of the six path rules; a space kind does not match.

### Provision entry

Closed key set; an unknown key is an error (`modules/provision/manifest.py:8-253`).

| Key                         | Type                                                       | Rule                                                                                                                                                                                                  |
| --------------------------- | ---------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`                      | non-empty string                                           | Optional. Unique within its scope across all sub-targets of one source.                                                                                                                               |
| `scope`                     | `workspace` \| `feature-environment` \| `feature-worktree` | Required.                                                                                                                                                                                             |
| `apply`                     | non-empty string, or non-empty array of non-empty strings  | Required.                                                                                                                                                                                             |
| `destroy`, `reset`, `clean` | same                                                       | Optional.                                                                                                                                                                                             |
| `required_services`         | array of string                                            | Only under `resource` and `data`.                                                                                                                                                                     |
| `project`                   | non-empty string                                           | Only with `scope = "feature-environment"`. In workspace configuration it must name a declared project repository; in an extension manifest the name is not checked against the declared repositories. |

The same schema serves `[provision]` in the workspace configuration, with `source` = `workspace`.

```text
# check order; the first fault found is the one reported           reference: modules/provision/manifest.py
per sub-target key:   unknown sub-target (known: dependency, resource, data)  ->  value is not an array
per entry:            not a table  ->  unknown keys  ->  scope missing  ->  scope invalid
                      ->  apply (missing, empty string, empty array, non-string or empty item, wrong type)
                      ->  destroy, reset, clean, each when present, same faults
                      ->  required_services outside resource/data  ->  required_services not an array of strings
                      ->  project with another scope  ->  project not a non-empty string  ->  project not declared
                      ->  name not a non-empty string  ->  name duplicated within its scope
```

### Service entry

Closed key set `name`, `cmd`, `command`, `image`, `scope`, `ports`, `target`
(`modules/service/ext_service_manifest.py:51`).

| Key       | Type                                 | Default               | Rule                                                                                                 |
| --------- | ------------------------------------ | --------------------- | ---------------------------------------------------------------------------------------------------- |
| `name`    | non-empty string                     | —                     | Required. Unique across the workspace and all extensions.                                            |
| `scope`   | `workspace` \| `feature-environment` | `feature-environment` |                                                                                                      |
| `cmd`     | string                               | unset                 |                                                                                                      |
| `command` | string                               | unset                 | Deprecated alias of `cmd`; a warning is logged when only `command` is set; `cmd` wins when both are. |
| `image`   | string                               | unset                 |                                                                                                      |
| `target`  | string                               | unset                 |                                                                                                      |
| `ports`   | array of string                      | `[]`                  |                                                                                                      |

The same schema serves `[[service]]` in the workspace configuration, with `source` = `workspace`.

```text
# check order                                                       reference: modules/service/ext_service_manifest.py
absent, or an empty value        ->  no service entries
not an array                     ->  NotTableList
per entry:  not a table  ->  unknown keys  ->  name missing or not a non-empty string  ->  scope invalid
            ->  cmd, command, image, target not a string  ->  ports not an array of strings
across sources (workspace first, then extensions): a name already seen, in the same or another source  ->  DuplicateName
```

## Flows

### Failure handling

```text
# engineering:manifest-validation, decision:agent-ready-operation/extension-management/manifest-policy
# reference: modules/workspace/extension_manifest.py:360-381
unknown top-level key, unknown key in [provides] / [implements] / [hooks]   -> ignored, no message
file unreadable, TOML parse failure, invalid `load`, malformed [provision],
malformed [[service]], malformed [excludes]                                 -> the manifest load fails: a repo error
                                                                               whose cause is the ManifestError
```

Each reader of a manifest decides what a failed load does:

| Reader                                                               | On a failed manifest load                                                                                                                                                 | Reference                                               |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------- |
| Context generation                                                   | The repository falls back to its repo-kind context defaults; the run continues. An excludes declaration failure is not absorbed: the run fails.                           | `modules/workspace/extension_manifest.py:34`            |
| Service definition collection (`service up`, `service ext-services`) | A warning on stderr naming the repository; that extension contributes no service definition; collection continues. Applies to every manifest failure, whatever its cause. | `modules/service/service_manifest_collector.py:127-132` |
| Provision handler collection (`provision`, `clean`)                  | A logged warning naming the repository; that extension contributes no provision handler; collection continues.                                                            | `modules/provision/provision_service.py:504-515`        |
| Hook dispatch                                                        | That repository's hook fails with the repo error; see the invocation contract.                                                                                            | `modules/workspace/extension_hook_service.py`           |
| Any other reader                                                     | The repo error propagates to that reader's caller; the owning command's spec states the result.                                                                           |                                                         |

A fault in the workspace configuration's own `[provision]` or `[[service]]` is not absorbed: it is a command error, exit
1 ([configuration model](../../../parallel-workspaces/workspace-configuration/spec/configuration-model.md), Unknown
keys).

## Open questions

- `skills_dir` and `agents_dir` are not type-checked by the reference tool, so a non-string value fails later, where the
  directory is used. Engineering pins the observable result with a recorded scenario in
  `epic:agent-ready-operation/harness-projection`.
