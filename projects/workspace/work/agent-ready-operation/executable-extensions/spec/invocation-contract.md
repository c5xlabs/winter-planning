# Invocation contract

What the binary gives every external program it runs — argument vector, working directory, standard input, environment —
and what it reads back. One row per seam; an extension author relies on nothing outside its row. Reference locations are
paths under `tools/winter-cli/src/winter_cli/` of the `winter-workspace` repository at `b57b020`. The declarations that
name these programs are the [extension manifest](../../extension-management/spec/extension-manifest.md) and the
[configuration model](../../../parallel-workspaces/workspace-configuration/spec/configuration-model.md); the `service`
provider seam is the [provider contract](../../../running-environments/service-orchestration/spec/provider-contract.md).

This is a contract toward other programs, so argument vectors, working directories, environment variable names and
values, and the two line protocols are fixed exactly. Texts the binary writes about a child — a failure message, a
result's `message` it composes itself — are its own wording (`decision:functional-parity`).

## Domain

Variable groups. Every child starts from a copy of the binary's own environment; a group adds to or overwrites it.

| Group          | Variables                                                                                                       | Values                                                                                                                                                                                            |
| -------------- | --------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Base five      | `WINTER_WORKSPACE_DIR`, `WINTER_EXT_DIR`, `WINTER_EXT_PREFIX`, `WINTER_EXT_CONFIG_DIR`, `WINTER_SERVICE_PREFIX` | Workspace root; extension checkout; resolved prefix; `<root>/.winter/config/<name>` or the entry's `config_dir`; the `service_prefix` key. Absolute paths. (`core/extension_invocation.py:16-50`) |
| Scope trio     | `WINTER_ENV`, `WINTER_ENV_INDEX`, `WINTER_PORT_BASE`                                                            | Env name; its index; `port_base(index)`. Decimal strings.                                                                                                                                         |
| Workspace port | `WINTER_WORKSPACE_PORT_BASE`                                                                                    | `port_base(0)`.                                                                                                                                                                                   |
| Bands          | every key of the applicable `env.*.vars` tables                                                                 | Feature scope: workspace band, then feature band, then `env.<name>` band, later winning. Workspace scope: workspace band only.                                                                    |
| Lint           | `WINTER_CLI`, `WINTER_LINT_SCOPE`, `WINTER_LINT_PATHS`                                                          | The binary's own resolved executable path; `all`, `repo`, `env`, or `changed`; newline-joined absolute paths.                                                                                     |
| Tracing        | `TRACEPARENT`, `TRACESTATE`                                                                                     | Only when tracing is on. Never passed to a launched service.                                                                                                                                      |

Workspace scope sets `WINTER_ENV=workspace` and `WINTER_ENV_INDEX=0` and has no `WINTER_PORT_BASE`
(`modules/workspace/env_provisioner.py:106-182`).

## Interfaces

### Seams

Standard input and the git prompt switch, for every row of the table below:

```text
# reference: container.py:145-177,213-215; core/internal/local_subprocess_runner.py:193-218;
#            modules/workspace/internal/subprocess_command_entry_runner.py:147
stdin                 inherited from the binary, so a child can read the terminal the binary was started on
                      closed (/dev/null) in two cases only:
                          an env band command entry, always
                          every child of an init run the dashboard starts in-process
GIT_TERMINAL_PROMPT   not set by the binary; a value already in its environment is inherited unchanged
                      set to 0 in one case only:
                          every child of an init run the dashboard starts in-process, the git calls of that
                          run, clone included, among them
```

A hook, probe, lint check, provision handler, setup command, service provider, or git call started by an ordinary
command therefore inherits standard input and receives no `GIT_TERMINAL_PROMPT` from the binary.

| Seam                                 | Declared by                   | argv                                               | cwd                                                      | Environment                                                                                                       | Read back                                                        |
| ------------------------------------ | ----------------------------- | -------------------------------------------------- | -------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| Hook `on_env_init`, `on_env_destroy` | manifest `[hooks]`            | script path alone                                  | `<root>/<env>`                                           | base five, workspace port, scope trio                                                                             | merged stream, by line; exit code                                |
| Hook `on_workspace_reconcile`        | manifest `[hooks]`            | script path alone                                  | workspace root                                           | base five, workspace port                                                                                         | same                                                             |
| Extension doctor probe               | manifest `doctor`             | script path alone                                  | workspace root                                           | base five                                                                                                         | captured: stdout as probe NDJSON, stderr separately; exit code   |
| Workspace doctor probe               | workspace key `doctor`        | script path alone                                  | workspace root                                           | `WINTER_WORKSPACE_DIR` only                                                                                       | same; source label `project`                                     |
| Extension lint check                 | manifest `lint`               | script path alone                                  | workspace root                                           | base five, lint                                                                                                   | captured: stdout as finding NDJSON, stderr separately; exit code |
| Workspace lint check                 | workspace key `lint`          | script path alone                                  | workspace root                                           | `WINTER_WORKSPACE_DIR`, lint                                                                                      | same                                                             |
| Extension provision handler          | manifest `[provision]`        | `sh -c <command>`, one per command                 | by scope: root, `<root>/<env>`, `<root>/<env>/<project>` | base five; scope trio on feature scopes. No workspace port, no bands.                                             | merged stream, by line; exit code                                |
| Workspace provision handler          | workspace `[provision]`       | same                                               | same                                                     | as above with `WINTER_EXT_DIR=<root>`, `WINTER_EXT_PREFIX=project`, `WINTER_EXT_CONFIG_DIR=<root>/.winter/config` | same                                                             |
| Repository setup command             | repository entry `cmd`        | `sh -c <command>`                                  | the checkout or worktree                                 | the quiet set below; for a `nested = true` project, added to the scrubbed environment below                       | merged stream, by line; exit code                                |
| Env band command entry               | `env.*.vars.<KEY>` table      | shell-split argv, or `sh -c` when `shell = true`   | workspace root                                           | the binary's environment, overlaid with the scope variables resolved so far that the entry may see                | captured: stdout parsed by `format`; stderr separately           |
| Service provider                     | manifest `[provides] service` | see the provider contract                          | workspace root                                           | see the provider contract                                                                                         | see the provider contract                                        |
| Nested workspace                     | project entry `nested = true` | the binary's own executable path, then the command | the nested root                                          | the nested set below                                                                                              | exit code; output passed through                                 |

Read-back modes (`core/internal/local_subprocess_runner.py:304-340`):

```text
merged stream   the child's stderr is redirected into its stdout; the binary reads one stream line by line as it
                is produced, so stderr lines and stdout lines arrive interleaved in the order the child wrote them
                each line becomes one output event: `cmd_output_line` for hooks and repository setup commands,
                `execution_output_line` for provision handlers; nothing distinguishes a stderr line from a stdout line
                # hooks: modules/workspace/extension_hook_service.py:247; setup commands: init_service.py:928;
                # provision handlers: modules/provision/execution_service.py:221
captured        stdout and stderr are collected separately and read after the child exits
```

Quiet set, added for repository setup commands (`modules/workspace/init_service.py:42-53,924`): `CI=1`, `TERM=dumb`,
`NO_COLOR=1`, `FORCE_COLOR=0`, `NPM_CONFIG_PROGRESS=false`, `PNPM_PROGRESS=false`, `PYTHONUNBUFFERED=1`,
`PIP_PROGRESS_BAR=off`, `DOTNET_NOLOGO=1`, `DOTNET_CLI_TELEMETRY_OPTOUT=1`.

Scrubbed environment (`modules/workspace/nested_env.py:31-55`): the binary's environment with every `WINTER_*` variable
removed except `WINTER_LOG_LEVEL` and `WINTER_OTEL_*`, and with `VIRTUAL_ENV` removed and its `bin` directory dropped
from `PATH`. A setup command of a `nested = true` project starts from it, so `WINTER_NESTED_CHAIN` is absent there
(`modules/workspace/init_service.py:920-924`); every other setup command starts from the binary's environment unchanged.

Nested set (`modules/workspace/nested_env.py:58-67`): the scrubbed environment, with `WINTER_NESTED_CHAIN` set to the
inherited chain extended with the current root. The child is the same executable the parent runs as, never a `PATH`
lookup.

### Execution policy

```text
# engineering:process-execution, decision:agent-ready-operation/executable-extensions/execution-policy
timeout:      none for hooks, probes, lint checks, provision handlers, setup commands
              30 s for an env band command entry; readiness waits are the provider contract's
output:       unbounded; streamed, not buffered, on the merged-stream rows
interaction:  the binary answers no prompt on a child's behalf; stdin is inherited except in the two cases
              Seams names, so a child may prompt on the terminal
inheritance:  the whole parent environment, plus the row's groups
trust:        a declared script is run when it resolves inside its declaring directory, exists, and is executable
```

Shell use. Four rows run a declared command line through `sh -c`; every other row executes an argument vector directly,
and no shell is inferred from other text.

| Row                                        | Command line declared by                 |
| ------------------------------------------ | ---------------------------------------- |
| Extension provision handler                | the extension, in its `winter-ext.toml`  |
| Workspace provision handler                | the operator, in workspace configuration |
| Repository setup command                   | the operator, in workspace configuration |
| Env band command entry with `shell = true` | the operator, in workspace configuration |

### Script preflight

Applies to hooks, probes, and lint checks. The declaring directory is the extension checkout, or the workspace root for
workspace keys.

| Condition                           | Hook                              | Probe or lint check                                                        |
| ----------------------------------- | --------------------------------- | -------------------------------------------------------------------------- |
| Path resolves outside the directory | repo error; the script is not run | one `fail` result; the script is not run                                   |
| File missing                        | same                              | same                                                                       |
| File not executable                 | same                              | same, with a remediation that names the path and how to make it executable |

### Hooks

```text
# product:lifecycle-contributions, decision:agent-ready-operation/executable-extensions/hook-policy
# reference: modules/workspace/extension_hook_service.py:116-255
order:      extensions in adoption order; within the lifecycle phase the owning command defines
skipped:    every hook, when adopt_extensions = "none"; a repository with no checkout or no manifest
absent:     a manifest that does not declare the hook counts as success, and nothing runs
events:     cmd_started with label "hook <hook name>", one cmd_output_line per line, cmd_completed with the exit code;
            on success, repo_action "hook_ran" carrying the hook name
success:    exit 0
failure:    a repo error for the declaring repository, reported as a repo_error event; how the owning command
            continues is that command's contract
```

```rust
// The typed cause of a hook's repo error.
pub enum HookError {
    Manifest(ManifestError),                                        // the manifest could not be loaded
    EscapesExtension { hook: String },                              // resolves outside the checkout; not run
    NotFound { hook: String, path: PathBuf },                       // not run
    NotExecutable { hook: String },                                 // not run
    Spawn { name: String, #[source] cause: std::io::Error },        // could not be started; the cause is the OS's
    Exit { name: String, code: i32 },                               // non-zero exit
}
```

### Probe protocol (`doctor`)

One JSON object per stdout line.

| Field         | Type                       | Required |
| ------------- | -------------------------- | -------- |
| `name`        | string                     | yes      |
| `status`      | `pass` \| `warn` \| `fail` | yes      |
| `message`     | string                     | no       |
| `remediation` | string                     | no       |

| Input                      | Result                                                                                                                   |
| -------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Line that is not JSON      | `warn`, name `doctor`; the message quotes the start of the line                                                          |
| JSON that is not an object | `warn`, name `doctor`                                                                                                    |
| Object without `name`      | `warn`, name `doctor`                                                                                                    |
| Unknown `status`           | `warn` under the given name; the message names the status given                                                          |
| Non-zero exit              | one more `fail`, name `doctor`; its message is the child's stderr, stripped, or names the exit code when stderr is empty |

### Finding protocol (`lint`)

One JSON object per stdout line.

| Field         | Type                       | Required | Note                             |
| ------------- | -------------------------- | -------- | -------------------------------- |
| `check`       | string                     | yes      | `name` is accepted in its place. |
| `status`      | `pass` \| `warn` \| `fail` | yes      |                                  |
| `message`     | string                     | no       |                                  |
| `file`        | string                     | no       |                                  |
| `line`        | integer or digit string    | no       | A boolean is ignored.            |
| `remediation` | string                     | no       |                                  |

| Input              | Result                                                                                                        |
| ------------------ | ------------------------------------------------------------------------------------------------------------- |
| Protocol violation | `warn`, check `lint`, under the same four cases as the probe protocol                                         |
| Non-zero exit      | one more `fail`, check `lint`; its message is the child's stderr, stripped, or names the exit code when empty |

### Conformance (`ext verify`)

```text
# decision:agent-ready-operation/executable-extensions/conformance-policy
`ext verify EXTENSIONS...` runs each operand's real entrypoint in the live workspace, with no isolation.
An operand is an installed extension's name or a local path to an extension directory; there is no slot operand.
The slot verified is `service`, the only one, and its checks are listed in the provider contract.
Any failed operand makes the exit code 1.
```

## Open questions

- The token grammar of band values (`${NAME}`, `${NAME+N}`, resolution to a fixed point) and the three command-entry
  output formats are owned by `epic:running-environments/scope-environment`; engineering specifies them there, with
  which resolved variables a command entry sees.
- One reference code path resolves an extension's configuration directory against the extension checkout rather than the
  workspace root. Engineering confirms with a recorded scenario whether any command can reach it; until then the
  base-five value above is the contract.
- Whether `ext new` is ported and what its scaffold writes is
  `decision:agent-ready-operation/executable-extensions/scaffold-command`. The product owner answers when
  `epic:agent-ready-operation/executable-extensions` is refined.
