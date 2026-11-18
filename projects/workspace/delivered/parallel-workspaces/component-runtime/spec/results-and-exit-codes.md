# Results, errors, and exit codes

How every invocation ends: the error taxonomy, which stream each class writes to in human and `--json` mode, every exit
code and its trigger, and interrupt behaviour. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of
the `winter-workspace` repository at `b57b020`; `click/` is the pinned Click 8.3.2 the reference tool runs on. The
command tree is [command-surface.md](./command-surface.md); stream and JSON conventions are
[output-formats.md](./output-formats.md); the process boundary, the prompt owner, and the signal handler are
[runtime.md](./runtime.md); the command span and its export are [tracing.md](./tracing.md).

Under `decision:functional-parity` the contract is the condition, the error kind with the values it carries, the stream,
and the exit code. The wording of every diagnostic is the binary's own; a reference text shown here is an example,
marked as such.

## Domain

```rust
// engineering:typed-outcomes, decision:parallel-workspaces/component-runtime/error-rendering
// One value per way an invocation ends. Commands return these and write only their RESULT (stdout) and the
// reporter and log lines of output-formats.md. The process boundary alone writes the final diagnostic of a
// Termination and maps its exit code. No variant is rendered into a JSON document.
// decision:parallel-workspaces/component-runtime/termination-payloads
// A variant that carries an error type holds it in a Box; the fields a variant holds inline are scalars or one
// String. Termination is at most four words wide, whatever its error types grow to.
pub enum Termination {
    Success,                           // exit 0
    Reported,                          // exit 1   the command's own result already carries the failure; nothing more is written
    Usage(Box<UsageError>),            // exit 2   the fault, the usage of the command at fault, where to ask for help
    BareGroup { help: String },        // exit 2   no command given at the root or in a group; the diagnostic is that node's help
    Command(Box<CommandError>),        // exit 1
    StatusCollection(Box<StatusCollectionError>), // exit 2   ws status only
    Config(Box<ConfigError>),          // exit 1
    Repo(Box<RepoError>),              // exit 1   message plus the detail fields that are present
    Space(Box<SpaceError>),            // exit 1
    Env(Box<EnvError>),                // exit 1
    Scaffold(Box<ScaffoldError>),      // exit 1   ext new only
    Aborted { interrupted: bool },     // exit 1   a declined or unanswerable confirmation
    Interrupted,                       // exit 130 service paths only; nothing written
    Provider(u8),                      // exit = the value, 1–255: the provider's own code, or 128 plus the number of
                                       //          the signal that ended it; service paths only; nothing written
    ProviderNotStarted(Box<ProviderStartError>), // exit 126 a provider entrypoint could not be executed; service paths only
    BrokenPipe,                        // exit 1   stdout closed by the reader; nothing written
    Internal(Box<InternalError>),      // exit 1
}
// The name of each variant is also the value of the command span's error.type: tracing.md, What the span carries.

// Usage and command errors. Each variant carries the values its message names; Display renders them.
#[derive(Debug, thiserror::Error)]
pub enum UsageError {
    Parse { command_path: Vec<String>, usage: Usage, fault: ParseFault },   // raised by the argument parser
    Rule  { command_path: Vec<String>, usage: Usage, rule: UsageRule },     // raised by a command; command-surface.md
}
// Usage: runtime.md, What a command receives. The parser fills it from the node at `command_path`; a command
// fills it from cx.invoked. Display of either variant is the three parts of Usage diagnostic below.
pub enum ParseFault {
    UnknownCommand { name: String },   UnknownOption { name: String },
    MissingArgument { metavar: String }, MissingOptionValue { option: String },
    InvalidValue { param: String, value: String, expected: String },
    UnexpectedArgument { values: Vec<String> },
    RepeatedOption { option: String },   // decision:parallel-workspaces/component-runtime/repeated-options
    NotUtf8 { argument: String },        // decision:parallel-workspaces/component-runtime/non-utf8-arguments
                                         // the argument with each invalid sequence replaced, for the error line
}

// A usage rule a command checks itself, after the parser accepted the command line: one variant per rule of
// command-surface.md, Combination rules, whose class there is usage (exit 2), carrying the values at fault.
// No variant holds a pre-rendered message.
pub enum UsageRule { /* the first: added, with UsageError::Rule, by the first command that refuses such a rule */ }

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

// The entrypoint that could not be executed and the operating system's error for it.
// reference: core/internal/local_subprocess_runner.py:267-280
#[derive(Debug, thiserror::Error)]
#[error("cannot start {}: {source}", .entrypoint.display())]
pub struct ProviderStartError { pub entrypoint: PathBuf, #[source] pub source: std::io::Error }

// decision:parallel-workspaces/component-runtime/internal-failure-rendering
// Display and std::error::Error are implemented by hand, not derived (repository-layout.md, Errors the foundation
// defines): Display is the source's for Unexpected and the message for Panic; source() is Unexpected's source.
#[derive(Debug)]
pub enum InternalError {
    Unexpected { source: Box<dyn std::error::Error + Send + Sync>,
                 backtrace: std::backtrace::Backtrace },
    Panic { message: String, backtrace: std::backtrace::Backtrace },
}
```

`Termination` is the `Err` type of nearly every fallible function of the library, and its error types grow with every
epic. Under `decision:parallel-workspaces/component-runtime/termination-payloads` its width does not follow them: a
variant that carries an error type holds it in a `Box`, and the only fields a variant holds inline are scalars or one
`String`. A variant added later follows the same rule, whatever the size of its error type on the day it is added. The
error types are written as their owning specs give them and hold no box for this purpose; the site that builds an ending
boxes the error there, and a site that reads one reads through the box. The functions and command files that return
`Result<_, Termination>` return it by value, and no function, module, or package carries a lint attribute or a lint
setting for the size of an error. Where a spec or a change writes
`Termination::Config(ConfigError::RootNotFound { .. })`, it names the variant and the error it holds; the box is not
written out.

`SpaceError`, `EnvError`, and `ScaffoldError` are the typed errors of `epic:agent-ready-operation/artifact-space`,
`epic:running-environments/scope-environment`, and `epic:agent-ready-operation/executable-extensions`; each owning spec
lists its variants. `ScaffoldError` exists only while `ext new` does
(`decision:agent-ready-operation/executable-extensions/scaffold-command`). `Interrupted`, `Provider`, and
`ProviderNotStarted` are returned only by the service commands of `epic:running-environments/service-orchestration`.

A variant exists in the code from the change whose behaviour first returns it. This epic's commands and runtime return
`Success`, `Reported`, `Usage` with `UsageError::Parse`, `BareGroup`, `Command`, `Config`, `BrokenPipe`, and `Internal`,
and build no other: `Aborted` arrives with the first command that prompts, `Interrupted` with the first command that
defers the interrupt, and `UsageError::Rule` with `UsageRule` with the first command that refuses a usage rule. The
SIGINT handler that is not deferred ends the process itself and returns nothing ([runtime.md](./runtime.md), Interrupts
and signals), so no ending stands for it.

Errors that never reach the boundary, each handled by its own layer: lint scope errors are converted to command errors
(`modules/lint/command.py:81`); provider status and describe parse errors are handled inside the service layer
(`modules/service/status_parser.py:41`, `describe_parser.py:23`); a failure inside one target of `ws init` or
`ws destroy` becomes that target's `repo_error` event and a `Reported` termination
(`modules/workspace/init_service.py:237-245`).

## Interfaces

### Error rendering

All diagnostics below go to stderr and are written by the process boundary. Stdout receives nothing from an error.
`--json` does not change any row. Every diagnostic but help and the aborted notice opens with one line that marks it as
an error and carries the message.

| Class                         | What the diagnostic carries                                                         | Exit | Reference                                                 |
| ----------------------------- | ----------------------------------------------------------------------------------- | ---- | --------------------------------------------------------- |
| Usage error                   | the fault, the usage line of the command at fault, the command that prints its help | 2    | `click/exceptions.py:72-92`, `cli.py:437-439`             |
| Bare group                    | that node's help                                                                    | 2    | `click/core.py:1579-1582,1826-1827`                       |
| Command error                 | the rule violated and the values at fault                                           | 1    | `cli.py:437-439`                                          |
| Config error                  | the file and key at fault                                                           | 1    | `cli.py:440-442`                                          |
| Repo error                    | the message, then each present detail field: command line, cwd, exit, stderr        | 1    | `cli.py:443-450`, `domain_model.py:323-337`               |
| Aborted                       | that the run was aborted                                                            | 1    | `cli.py:434-436`                                          |
| `ws status` collection error  | the underlying failure                                                              | 2    | `modules/workspace/handlers/workspace_handler.py:298-303` |
| `space` invalid kind          | the kind given                                                                      | 1    | `modules/space/command.py:58-63`                          |
| `env` unknown scope / compute | the scope given, or the compute failure                                             | 1    | `modules/workspace/env_command.py:72-83`                  |
| `ext new` existing directory  | the directory                                                                       | 1    | `modules/ext/handler.py:59-60`                            |
| Provider not started          | the entrypoint and the operating system's error                                     | 126  | `core/internal/local_subprocess_runner.py:267-280`        |
| Internal failure              | the underlying error; see Internal failures                                         | 1    | `cli.py:430-450` has no handler                           |

The reference tool prefixes usage and command errors `Error:` and configuration and repository errors `error:`; the
binary uses one marker for all. A repo error's exit detail distinguishes a signal death (negative `exit`) from an exit
status.

### Usage diagnostic

```text
# product:automation-boundary — reference: click/exceptions.py:72-92, click/core.py:98-113
parts, in this order, one per line
  1. the error line: the marker and the fault (ParseFault or UsageRule, with the values it carries)
  2. Usage.line: the usage line of the node at `command_path`, as that node's --help prints it
  3. Usage.help_command: the command line that prints that node's help
the node at fault
  the deepest node the tokens resolved before the fault: the root for a root option fault or an unknown top-level
  command, the group for an unknown command of a group, the leaf for a fault in a command's own arguments
  `command_path` is the command path of runtime.md, The parse and the run
a usage error a command raises (UsageError::Rule) renders the same three parts, from cx.invoked
ParseFault::UnexpectedArgument renders each value quoted, so an empty value and a value holding a line break stay
on the error line; ParseFault::NotUtf8 renders its argument the same way
a bare root or group is not a usage diagnostic: its diagnostic is the node's whole help
```

### Internal failures

```text
# decision:parallel-workspaces/component-runtime/internal-failure-rendering
any failure with no class above, and any panic:
    stderr: one error line carrying the underlying error or the panic message      exit 1      stdout: nothing further
backtrace:
    captured with std::backtrace::Backtrace::capture(), which is active only when RUST_BACKTRACE or
    RUST_LIB_BACKTRACE is set to a value other than "0"
    when captured, it is written to stderr after the error line; otherwise nothing follows
panics:
    the binary installs its own panic hook and unwinds to the boundary; exit code 101 never occurs
    panic = "unwind" in every profile; no library crate catches a panic
    runtime.md, The process boundary, gives the hook and the boundary's steps
in-band internal failures (a target of `ws init` / `ws destroy`):
    stderr: a diagnostic naming the target, then the backtrace under the same rule
    event:  repo_error for that target; its `error` value is message text
    the target fails with target_completed success=false; the remaining targets still run; the run exits 1
```

### A stdout that cannot be written

```text
# reference: click/core.py:1425-1431 (EPIPE), click/utils.py:267-276 (no stream)
reader closed the pipe        the write fails with BrokenPipe      Termination::BrokenPipe: exit 1, nothing written
any other failed write        a full device, an I/O error          Termination::Internal: the error line, exit 1
failed final flush            the boundary flushes stdout after `run`; a failure is classed as a failed write, and
                              replaces the ending only when that ending's exit code is 0
no stdout at all (`>&-`)      writes are accepted and dropped       the command's own ending and exit code
a state-changing command has made its change before it writes its result: a failed write does not undo it
a failed write to stderr      ignored; the ending and its exit code stand
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

| Code    | Meaning              | Triggers                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| ------- | -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `0`     | Success              | Normal completion; `--help`; `--version`. Also these no-ops: `provision`, `clean`, `ws destroy` with glob patterns matching no env; `lint` with scopes matching nothing; `ws connect`, `disconnect`, `reset`, `clean` matching no worktree; `ws prune` with nothing removable. `capabilities` and `agents` with a broken slot or cell. `service catalog` with no provider.                                                                                                                                                                                                                                                                                                                                            |
| `1`     | Failure              | Command error; config error; repo error; aborted (declined or end-of-input confirmation, or interrupt outside the service paths); refusal or failure of `ws init`, `destroy`, `restack`, `checkout`, `reset`, `clean`, `fetch`, `pull`, `update`, `merge`, `push`; `ext verify` with any failed operand; `ext new` into an existing directory; `service up --wait` readiness timeout, also after a provider exit of 0; any failed `doctor` probe; any failed `lint` finding; `provision` or `clean` status other than `ok`; `space` invalid kind; `env` unknown scope or compute error; `ws status` when dirty or drifted (below); service-layer internal failure; a stdout that cannot be written; internal failure. |
| `2`     | Usage error          | Unknown command or option; missing argument or option value; invalid choice; unexpected argument; a repeated option; an argument that is not UTF-8; a value given to a flag; bare root or group; invalid `service logs --tail`, `--since`, `--until`; `ws update --force` without `--freeze`; `ws update --freeze --autostash`; `ext new` with an unknown capability slot; `ws connect`, `reset`, `restack` with no token at all. Also `ws status` when collecting the snapshot fails for any reason, a pattern that matches no worktree included.                                                                                                                                                                    |
| `126`   | Provider not started | A service provider entrypoint could not be executed: it is missing, not executable, or the operating system refused to start it. Produced by the binary in place of a provider code; the provider never ran. `service up`, `down`, `restart`, `describe`, `logs`, `status` only.                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| `130`   | Interrupted          | Interrupt during `service logs`, `service status`, or `service up --wait` readiness polling. Nowhere else.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `1–255` | Provider passthrough | `service up`, `down`, `restart`, `describe`, `logs`, `status` return the exit code of a provider entrypoint that ran and exited; across a fan-out, the first non-zero code. A provider ended by a signal gives 128 plus the signal number (`decision:parallel-workspaces/component-runtime/provider-signal-exit`); the reference tool gives 256 minus it (`modules/service/handler.py:68-73`). Not passed through: the 126 above and the readiness timeout.                                                                                                                                                                                                                                                           |

No other code is produced: `101` never, and the binary itself never exits by re-raising a signal.

```text
# decision:parallel-workspaces/component-runtime/exit-code-overloads
# reference: modules/workspace/handlers/workspace_handler.py:291-312,1363-1410
ws status
  2  collecting the snapshot failed; a PATTERN that matches no worktree is such a failure
     (modules/workspace/workspace_snapshot_service.py:213-215)
  1  with PATTERNS:    any matched worktree is dirty; source-checkout drift, standalone divergence, orphans, and
                       configuration drift are shown and do not change the code
     without PATTERNS: any worktree is dirty; or any project source checkout is behind or ahead of origin, dirty,
                       or drifted; or any standalone is behind, ahead, or dirty; or any orphan exists; or the
                       configuration has drifted
     a nested workspace with a dirty env, or one whose state cannot be read, counts as a dirty worktree
  0  otherwise
service *:   a provider's code wins over the categories above, except as the Provider passthrough row lists
             SIGTERM ending the provider gives 143, SIGKILL 137, SIGINT 130: the value alone does not tell a
             provider ended by SIGINT from the Interrupted row
```

Reference for the triggers: `cli.py:431-450`; `modules/workspace/handlers/init_handler.py:50`,
`destroy_handler.py:55-76`, `restack_handler.py:77,86`, `workspace_handler.py:426-794,1363-1410`;
`modules/ext/handler.py:45,60`; `modules/service/handler.py:68-121`, `service_dispatch_service.py:62-247`,
`service_fan_out_service.py:157-187`; `core/internal/local_subprocess_runner.py:267-280`;
`modules/doctor/doctor_service.py:26`; `modules/lint/models.py:223`; `modules/provision/provision_service.py:142`;
`modules/space/command.py:63`; `modules/workspace/env_command.py:76,83`.

### Interrupts and prompts

| Event                                                                | Result                                | Exit   | Reference                                                                                                                                 |
| -------------------------------------------------------------------- | ------------------------------------- | ------ | ----------------------------------------------------------------------------------------------------------------------------------------- |
| Interrupt (SIGINT) outside the service paths                         | aborted at once; a notice on stderr   | `1`    | `cli.py:434-436`                                                                                                                          |
| Interrupt during `service logs`, `service status`, readiness polling | interrupted; nothing required         | `130`  | `modules/service/service_logs_service.py:203,351`, `service_status_matrix_service.py:366-369,536-539`, `service_status_service.py:97,124` |
| Interrupt during `service up`, `down`, `restart` dispatch            | aborted, as outside the service paths | `1`    | no handler in `modules/service/handler.py:66-73`                                                                                          |
| Confirmation declined                                                | aborted before any mutation           | `1`    | `cli.py:434-436`                                                                                                                          |
| Confirmation reads end of input                                      | aborted before any mutation           | `1`    | same                                                                                                                                      |
| SIGTERM, SIGHUP, SIGQUIT                                             | the default disposition; no notice    | signal | no handler in the reference tool                                                                                                          |

[runtime.md](./runtime.md), Interrupts and signals, gives the handler; Confirmation gives the prompt's reading rules.

Confirmation prompts. Each is written to stdout after a preview naming what will be affected, defaults to "no", reads
its answer from stdin whether or not stdin is a terminal, and is skipped by `--force` or `--dry-run`:

| Command           | Prompts when                                             | Preview names                        | Under `--json`                                                            |
| ----------------- | -------------------------------------------------------- | ------------------------------------ | ------------------------------------------------------------------------- |
| `ws destroy`      | more than one PATTERN is given, or any PATTERN is a glob | the count and names of the envs      | same prompt, as plain text on stdout                                      |
| `ws reset --hard` | more than one worktree is targeted                       | the count and `<env>/<repo>` of each | same prompt, as plain text on stdout                                      |
| `ws clean`        | the preview finds at least one file to remove            | the files per worktree               | never reached: `--json` without `--force` or `--dry-run` is refused first |
| `ws prune`        | at least one orphan is safe to remove                    | each orphan, and the removable count | prompt skipped; safe orphans are removed without asking                   |

`ws prune` with orphans that are all blocked lists them, removes nothing, and exits 0 without prompting. Reference:
`modules/workspace/handlers/destroy_handler.py:59-61`, `workspace_handler.py:497-502,585-591,960-983`.

A run that ends through a returned `Termination` exports its command span like any other, a declined confirmation and
the exit 130 of a service path included: the exit path is single, and the exporter wait is capped by
`decision:observability-shutdown`. A run that SIGINT aborts outside the service paths exports none, under
`decision:parallel-workspaces/component-runtime/interrupt-trace-export`; the reference tool exports there
(`cli.py:451-455`).

## Flows

```text
# reference: cli.py:394-455 (process boundary), 366-378 (tracing environment), 211-228 (command span),
#            324-332 (logging), 150-162 (error.type); click/core.py:1866-1874 (stage order)
# decision:parallel-workspaces/component-runtime/single-parse-pass
start
  -> install the SIGINT handler and the panic hook
  -> snapshot the arguments, the working directory, and the environment; bind the streams
  -> read the tracing environment once                                       # tracing.md, Domain
       WINTER_LAUNCH_TIME is left out of every child's environment, set or not, valid or not; a valid value becomes
       the command span's start instant, and any other value, or none, starts the span when it is opened
  -> default GIT_SSH_COMMAND for children when unset
  -> parse the whole command line in one pass, left to right                 # runtime.md, The parse and the run
     an argument that is not UTF-8, anywhere on the line -> Usage -> exit 2, at the stage the tokens before it reached
     (decision:parallel-workspaces/component-runtime/non-utf8-arguments; command-surface.md, Parsing rules)
     root stage: the ending is decided before any command name resolved      # no span, no logging, no record
       root option fault            -> Usage            -> exit 2
       --version, root --help       -> stdout           -> exit 0
       no command given, with or without root options   -> root help on stderr   -> exit 2
       unknown top-level command    -> Usage            -> exit 2
     command stage: a top-level command name resolved
       -> resolve logging (-v, else WINTER_LOG_LEVEL) and the root option values; write the runtime's Debug record
       -> open the command span, named by the command path the tokens resolved
       parse failure, unknown subcommand included   -> Usage            -> exit 2
       --help of a command or subgroup              -> stdout           -> exit 0
       bare subgroup                                -> help on stderr   -> exit 2
     where --help or --version and a fault stand at one node, the one that stands first decides
     (decision:parallel-workspaces/component-runtime/help-beside-fault; command-surface.md, Parsing rules)
  -> unreadable working directory   -> Internal         -> exit 1
  -> run the command; workspace located and configuration loaded on first need
       no workspace root            -> Config           -> exit 1
       malformed configuration      -> Config           -> exit 1
       command writes its result, returns a Termination
       panic                        -> Internal         -> exit 1
  -> boundary flushes stdout; the ending is final
  -> boundary writes the diagnostic for the Termination, if any
  -> close the command span; it carries `error.type`, the ending's name, unless the invocation ends with exit 0
  -> export the trace when a command span was opened (bounded wait)
  -> exit with the mapped code
SIGINT, not deferred, at any point after the first step: the aborted notice, exit 1; nothing above runs further
```

The stages are properties of the ending, not two parser runs: the parser reads the command line once, and the runtime
derives the stage from the command path the tokens resolved. The parser writes no log record, so the order "logging,
then the command's own arguments" of the reference tool has no observable trace beyond the endings listed.

Exits and the command span:

| Exit                                                                                 | Span                                                              |
| ------------------------------------------------------------------------------------ | ----------------------------------------------------------------- |
| `--version`; root `--help`; root option fault; no command; unknown top-level command | none is opened, and nothing is exported                           |
| Usage error in a command's own arguments; unknown subcommand; bare subgroup          | inside the span; exported with `error.type`                       |
| `--help` of a command or subgroup                                                    | inside the span; exported without `error.type`                    |
| Every exit of a running command, internal failures included                          | inside the span; exported, with `error.type` unless the exit is 0 |
| SIGINT that is not deferred                                                          | the span, when open, is dropped; nothing is exported              |

Outside a workspace, and with a malformed `.winter/config.toml`, every root-stage ending, every `--help`, every bare
group, and every usage error ends as listed: none of them locates a workspace. A command that needs the workspace ends
with a config error when it first asks for it (reference: `cli_context.py:13-31`,
`config/internal/cwd_workspace_locator.py:26-33`). The value that holds the located root and the loaded configuration of
one invocation is
[configuration-model.md](../../../../work/parallel-workspaces/workspace-configuration/spec/configuration-model.md)'s.

## Open questions

- What the service layer writes to stderr before it ends with 126 is not recorded here; engineering states it in the
  provider contract when `epic:running-environments/service-orchestration` is refined.
- Whether a service command tells a provider ended by SIGINT (130 by the passthrough rule) from its own interrupt (130)
  in anything but the span's `error.type` is not recorded here; engineering states it in the provider contract when
  `epic:running-environments/service-orchestration` is refined.
- `Aborted { interrupted }`: no path specified so far produces `true`, because an interrupt that is not deferred ends
  the process in the handler. Engineering keeps or drops the field when the first prompting command is refined, in the
  epic that owns it (`epic:parallel-workspaces/environment-lifecycle` for `ws destroy`).
