# Results, errors, and exit codes

How every invocation ends: the error taxonomy, which stream each class writes to in human and `--json` mode, every exit
code and its trigger, and interrupt behaviour. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
the `winter-workspace` repository at `b57b020`. The command tree is [command-surface.md](./command-surface.md); stream
and JSON conventions are [output-formats.md](./output-formats.md).

Under `decision:functional-parity` the contract is the condition, the error kind with the values it carries, the stream,
and the exit code. The wording of every diagnostic is the binary's own; a reference text shown here is an example,
marked as such.

## Domain

```rust
// engineering:typed-outcomes, decision:parallel-workspaces/component-runtime/error-rendering
// One value per way an invocation ends. Commands return these and write only their RESULT (stdout) and the
// reporter and log lines of output-formats.md. The process boundary alone writes the final diagnostic of a
// Termination and maps its exit code. No variant is rendered into a JSON document.
pub enum Termination {
    Success,                           // exit 0
    Reported,                          // exit 1   the command's own result already carries the failure; nothing more is written
    Usage(UsageError),                 // exit 2   usage plus the fault
    Command(CommandError),             // exit 1
    StatusCollection(StatusCollectionError), // exit 2   ws status only
    Config(ConfigError),               // exit 1
    Repo(RepoError),                   // exit 1   message plus the detail fields that are present
    Space(SpaceError),                 // exit 1
    Env(EnvError),                     // exit 1
    Scaffold(ScaffoldError),           // exit 1   ext new only
    Aborted { interrupted: bool },     // exit 1
    Interrupted,                       // exit 130 service paths only; nothing written
    Provider(u8),                      // exit = the provider's own code, service paths only; nothing written
    BrokenPipe,                        // exit 1   stdout closed by the reader; nothing written
    Internal(InternalError),           // exit 1
}

// Usage and command errors. Each variant carries the values its message names; Display renders them.
#[derive(Debug, thiserror::Error)]
pub enum UsageError {
    Parse { command_path: Vec<String>, fault: ParseFault },   // raised by the argument parser
    Rule  { command_path: Vec<String>, rule: UsageRule },     // raised by a command; command-surface.md
}
pub enum ParseFault {
    UnknownCommand { name: String },   UnknownOption { name: String },
    MissingArgument { metavar: String }, MissingOptionValue { option: String },
    InvalidValue { param: String, value: String, expected: String },
    UnexpectedArgument { values: Vec<String> },
}

#[derive(Debug, thiserror::Error)]
pub enum CommandError {                // one variant per module; each wraps that module's own thiserror enum
    #[error(transparent)] Selection(#[from] SelectionError),     // patterns, modes, flag combinations
    #[error(transparent)] Catalog(#[from] CatalogError),         // repo add / remove
    #[error(transparent)] Provision(#[from] ProvisionCommandError),
    #[error(transparent)] Service(#[from] ServiceCommandError),  // collisions, malformed workspace [[service]]
    #[error(transparent)] Lint(#[from] LintScopeError),
    // … one per remaining module, added by the epic that owns it
}
// A module error enum has one variant per rule in command-surface.md "Combination rules" or its own epic's spec,
// carrying the values at fault (flag names, the pattern, the repo kind and name). No variant holds a
// pre-rendered message.

// Any failure while `ws status` collects its snapshot: a command error, a config error, a repo error, or an
// unexpected one. The reference tool catches every exception there (workspace_handler.py:298-303).
#[derive(Debug, thiserror::Error)]
#[error("{source}")]
pub struct StatusCollectionError { #[source] pub source: Box<dyn std::error::Error + Send + Sync> }

// ConfigError: variants in ../../workspace-configuration/spec/configuration-model.md, Domain.

#[derive(Debug, thiserror::Error)]
#[error("{message}")]
pub struct RepoError {                 // reference: modules/workspace/models/domain_model.py:288
    pub message: RepoMessage,          // typed cause
    pub program: Option<String>,       // each detail field is reported only when present
    pub subcommand: Option<String>,
    pub args: Vec<String>,
    pub cwd: Option<PathBuf>,
    pub exit: Option<i32>,             // negative = killed by that signal number
    pub stderr: Option<String>,
}
// RepoMessage is an enum: a git failure (operation, repo), a manifest failure (extension-manifest.md), a hook
// failure (invocation-contract.md), a partial clean, an excludes declaration failure, a capability binding
// failure with its version-compat cause. Callers match the variant, never the text.
//   reference kinds: PartialClean (domain_model.py:340), ExcludesDeclaration (extension_manifest.py:34),
//   CapabilityBinding (capability/models.py:120), VersionCompat (capability/version_compat.py:6).

// decision:parallel-workspaces/component-runtime/internal-failure-rendering
#[derive(Debug, thiserror::Error)]
pub enum InternalError {
    #[error("{source}")]
    Unexpected { #[source] source: Box<dyn std::error::Error + Send + Sync>,
                 backtrace: std::backtrace::Backtrace },
    #[error("{message}")]
    Panic { message: String, backtrace: std::backtrace::Backtrace },
}
```

`SpaceError`, `EnvError`, and `ScaffoldError` are the typed errors of `epic:agent-ready-operation/artifact-space`,
`epic:running-environments/scope-environment`, and `epic:agent-ready-operation/executable-extensions`; each owning spec
lists its variants. `ScaffoldError` exists only while `ext new` does
(`decision:agent-ready-operation/executable-extensions/scaffold-command`).

Errors that never reach the boundary, each handled by its own layer: lint scope errors are converted to command errors
(`modules/lint/command.py:81`); provider status and describe parse errors are handled inside the service layer
(`modules/service/status_parser.py:41`, `describe_parser.py:23`); a failure inside one target of `ws init` or
`ws destroy` becomes that target's `repo_error` event and a `Reported` termination
(`modules/workspace/init_service.py:237-245`).

## Interfaces

### Error rendering

All diagnostics below go to stderr and are written by the process boundary. Stdout receives nothing from an error.
`--json` does not change any row.

| Class                         | What the diagnostic carries                                                  | Exit | Reference                                                 |
| ----------------------------- | ---------------------------------------------------------------------------- | ---- | --------------------------------------------------------- |
| Usage error                   | the usage of the command at fault, and the fault                             | 2    | `cli.py:437-439`                                          |
| Bare group                    | that group's help                                                            | 2    | —                                                         |
| Command error                 | the rule violated and the values at fault                                    | 1    | `cli.py:437-439`                                          |
| Config error                  | the file and key at fault                                                    | 1    | `cli.py:440-442`                                          |
| Repo error                    | the message, then each present detail field: command line, cwd, exit, stderr | 1    | `cli.py:443-450`, `domain_model.py:323-337`               |
| Aborted                       | that the run was aborted                                                     | 1    | `cli.py:434-436`                                          |
| `ws status` collection error  | the underlying failure                                                       | 2    | `modules/workspace/handlers/workspace_handler.py:298-303` |
| `space` invalid kind          | the kind given                                                               | 1    | `modules/space/command.py:58-63`                          |
| `env` unknown scope / compute | the scope given, or the compute failure                                      | 1    | `modules/workspace/env_command.py:72-83`                  |
| `ext new` existing directory  | the directory                                                                | 1    | `modules/ext/handler.py:59-60`                            |
| Internal failure              | the underlying error; see Internal failures                                  | 1    | `cli.py:430-450` has no handler                           |

The reference tool prefixes usage and command errors `Error:` and configuration and repository errors `error:`; the
binary need not. A repo error's exit detail distinguishes a signal death (negative `exit`) from an exit status.

```text
# decision:parallel-workspaces/component-runtime/git-failure-rendering
A failed git call is rendered once: by the repo error when it reaches the boundary, or by the reporter line or
`repo_error` event when a command handles it per repository.
With `-v` or a valid `WINTER_LOG_LEVEL` the failure is also an ordinary log record on stderr.
The reference tool writes that record as a bare stderr line in every mode
(modules/workspace/internal/repo_error_factory.py:103,136,178,240); the binary writes nothing extra when no
logging switch is set.
```

### Internal failures

```text
# decision:parallel-workspaces/component-runtime/internal-failure-rendering
any failure with no class above, and any panic:
    stderr: a diagnostic carrying the underlying error or the panic message      exit 1      stdout: nothing further
backtrace:
    captured with std::backtrace::Backtrace::capture(), which is active only when RUST_BACKTRACE or
    RUST_LIB_BACKTRACE is set to a value other than "0"
    when captured, it is written to stderr after the diagnostic; otherwise nothing follows
panics:
    the binary installs its own panic hook and unwinds to the boundary; exit code 101 never occurs
    panic = "unwind" in every profile; no library crate catches a panic
in-band internal failures (a target of `ws init` / `ws destroy`):
    stderr: a diagnostic naming the target, then the backtrace under the same rule
    event:  repo_error for that target; its `error` value is message text
    the target fails with target_completed success=false; the remaining targets still run; the run exits 1
closed stdout pipe:
    exit 1; nothing is written
```

### No JSON error envelope

```text
# product:automation-boundary, decision:parallel-workspaces/component-runtime/error-rendering
--json changes how a command writes its RESULT. It changes nothing about errors:
  error -> plain text on stderr; stdout empty, or an event stream that stops early
  consumers decide success from the exit code
```

Failure is reported in-band only where the command's own document or event vocabulary carries it:

| Carrier               | Commands                                                                                         | Field or event                                                                                                                                               |
| --------------------- | ------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Terminal NDJSON event | `ws fetch`, `pull`, `update`, `merge`, `reset`, `clean`, `init`, `destroy`, `provision`, `clean` | `fetch_completed`, `pull_completed`, `merge_completed`, `reset_completed`, `clean_completed`, `target_completed`, `finished`, carrying `success` or `status` |
| Per-item NDJSON event | `ws init`, `ws destroy`, `provision`, `clean`                                                    | `repo_error`, `execution_error`                                                                                                                              |
| Document field        | `ws checkout`                                                                                    | `aborted`                                                                                                                                                    |
| Document field        | `ws restack`                                                                                     | `refusals`, `conflict`, `failure`                                                                                                                            |
| Document field        | `ext verify`                                                                                     | `setup_failure`, `any_failed`                                                                                                                                |
| Document field        | `ws prune`                                                                                       | per-orphan `action: "error"` with `error`                                                                                                                    |
| Document field        | `service catalog`                                                                                | `error` key, with exit 0, when no provider resolves                                                                                                          |

### Exit codes

| Code    | Meaning              | Triggers                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| ------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`     | Success              | Normal completion; `--help`; `--version`. Also these no-ops: `provision`, `clean`, `ws destroy` with glob patterns matching no env; `lint` with scopes matching nothing; `ws connect`, `disconnect`, `reset`, `clean` matching no worktree; `ws prune` with nothing removable. `capabilities` and `agents` with a broken slot or cell. `service catalog` with no provider.                                                                                                                                                                                                                                                                                      |
| `1`     | Failure              | Command error; config error; repo error; aborted (declined or end-of-input confirmation, or interrupt outside the service paths); refusal or failure of `ws init`, `destroy`, `restack`, `checkout`, `reset`, `clean`, `fetch`, `pull`, `update`, `merge`, `push`; `ext verify` with any failed operand; `ext new` into an existing directory; `service up --wait` timeout; any failed `doctor` probe; any failed `lint` finding; `provision` or `clean` status other than `ok`; `space` invalid kind; `env` unknown scope or compute error; `ws status` when dirty or drifted (below); service-layer internal failure; a closed stdout pipe; internal failure. |
| `2`     | Usage error          | Unknown command or option; missing argument; invalid choice; bare group; invalid `service logs --tail`, `--since`, `--until`; `ws update --force` without `--freeze`; `ws update --freeze --autostash`; `ext new` with an unknown capability slot; `ws connect`, `reset`, `restack` with no token at all. Also `ws status` when collecting the snapshot fails for any reason.                                                                                                                                                                                                                                                                                   |
| `130`   | Interrupted          | Interrupt during `service logs`, `service status`, or `service up --wait` readiness polling. Nowhere else.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `1–255` | Provider passthrough | `service up`, `down`, `restart`, `describe`, `logs`, `status` return the provider entrypoint's exit code unmodified; across a fan-out, the first non-zero code.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

```text
# decision:parallel-workspaces/component-runtime/exit-code-overloads
# reference: modules/workspace/handlers/workspace_handler.py:291-312,1363-1410
ws status
  2  collecting the snapshot failed
  1  with PATTERNS:    any matched worktree is dirty; source-checkout drift, standalone divergence, orphans, and
                       configuration drift are shown and do not change the code
     without PATTERNS: any worktree is dirty; or any project source checkout is behind or ahead of origin, dirty,
                       or drifted; or any standalone is behind, ahead, or dirty; or any orphan exists; or the
                       configuration has drifted
     a nested workspace with a dirty env, or one whose state cannot be read, counts as a dirty worktree
  0  otherwise
service *:   the provider's code wins over the categories above
```

Reference for the triggers: `cli.py:431-450`; `modules/workspace/handlers/init_handler.py:50`,
`destroy_handler.py:55-76`, `restack_handler.py:77,86`, `workspace_handler.py:426-794,1363-1410`;
`modules/ext/handler.py:45,60`; `modules/service/handler.py:68-121`, `service_dispatch_service.py:62-247`,
`service_fan_out_service.py:157-187`; `modules/doctor/doctor_service.py:26`; `modules/lint/models.py:223`;
`modules/provision/provision_service.py:142`; `modules/space/command.py:63`; `modules/workspace/env_command.py:76,83`.

### Interrupts and prompts

| Event                                                                | Result                        | Exit  | Reference                                                                                                                                 |
| -------------------------------------------------------------------- | ----------------------------- | ----- | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Interrupt (SIGINT) outside the service paths                         | aborted; a notice on stderr   | `1`   | `cli.py:434-436`                                                                                                                          |
| Interrupt during `service logs`, `service status`, readiness polling | interrupted; nothing required | `130` | `modules/service/service_logs_service.py:203,351`, `service_status_matrix_service.py:366-369,536-539`, `service_status_service.py:97,124` |
| Confirmation declined                                                | aborted before any mutation   | `1`   | `cli.py:434-436`                                                                                                                          |
| Confirmation reads end of input                                      | aborted before any mutation   | `1`   | same                                                                                                                                      |

Confirmation prompts. Each is written to stdout after a preview naming what will be affected, defaults to "no", and is
skipped by `--force` or `--dry-run`:

| Command           | Prompts when                                             | Preview names                        | Under `--json`                                                            |
| ----------------- | -------------------------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------- |
| `ws destroy`      | more than one PATTERN is given, or any PATTERN is a glob | the count and names of the envs      | same prompt, as plain text on stdout                                      |
| `ws reset --hard` | more than one worktree is targeted                       | the count and `<env>/<repo>` of each | same prompt, as plain text on stdout                                      |
| `ws clean`        | the preview finds at least one file to remove            | the files per worktree               | never reached: `--json` without `--force` or `--dry-run` is refused first |
| `ws prune`        | at least one orphan is safe to remove                    | each orphan, and the removable count | prompt skipped; safe orphans are removed without asking                   |

`ws prune` with orphans that are all blocked lists them, removes nothing, and exits 0 without prompting. Reference:
`modules/workspace/handlers/destroy_handler.py:59-61`, `workspace_handler.py:497-502,585-591,960-983`.

An interrupted or aborted run exports its trace like any other: the exit path is single, and the exporter wait is capped
by `decision:observability-shutdown`.

## Flows

```text
# reference: cli.py:394-455 (process boundary), 366-378 (tracing environment), 211-228 (command span),
#            324-332 (logging), 150-162 (error.type)
start
  -> install the panic hook
  -> read the tracing environment once
       WINTER_LAUNCH_TIME is taken out of the process environment as it is read, set or not, valid or not,
       so no child process inherits it; a valid value becomes the command span's start instant, and any
       other value, or none, starts the span when it is opened
  -> default GIT_SSH_COMMAND when unset
  -> parse the root options and find the top-level command            # no span is open yet
       root option fault            -> Usage            -> exit 2
       --version, root --help       -> stdout           -> exit 0
       no command given             -> help on stderr   -> exit 2
       unknown top-level command    -> Usage            -> exit 2
  -> open the command span, named by the command path, as soon as the top-level command name resolves
  -> configure logging (-v, else WINTER_LOG_LEVEL)
  -> parse the command's own arguments                                # inside the span
       parse failure, unknown subcommand included   -> Usage            -> exit 2
       --help of a command or subgroup              -> stdout           -> exit 0
       bare subgroup                                -> help on stderr   -> exit 2
  -> run the command; workspace located and configuration loaded on first need
       no workspace root            -> Config           -> exit 1
       malformed configuration      -> Config           -> exit 1
       command writes its result, returns a Termination
       panic                        -> Internal         -> exit 1
  -> close the command span; it carries `error.type` unless the invocation ends with exit 0
  -> boundary writes the diagnostic for the Termination, if any
  -> export the trace when a command span was opened (bounded wait)
  -> exit with the mapped code
```

Exits and the command span:

| Exit                                                                                 | Span                                                              |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `--version`; root `--help`; root option fault; no command; unknown top-level command | none is opened, and nothing is exported                           |
| Usage error in a command's own arguments; unknown subcommand; bare subgroup          | inside the span; exported with `error.type`                       |
| `--help` of a command or subgroup                                                    | inside the span; exported without `error.type`                    |
| Every exit of a running command, interrupts and internal failures included           | inside the span; exported, with `error.type` unless the exit is 0 |

## Open questions

- The exit status on SIGTERM and SIGHUP is not recorded for the reference tool. Engineering records it with the
  foundation's signal scenarios.
- Whether `ws status` with a pattern that matches no worktree is a collection failure (exit 2) or a clean result
  (exit 0) in the reference tool is not confirmed from source. Engineering records it with the `ws status` scenarios.
