# Command surface

The complete command tree of the `winter-workspace` binary. It mirrors the reference tool's `winter …` tree node for
node under `decision:command-vocabulary`; reference locations are paths under `tools/winter-cli/src/winter_cli/` of the
`winter-workspace` repository at `b57b020`. Exit codes and error kinds are in
[results-and-exit-codes.md](./results-and-exit-codes.md); stream and JSON conventions in
[output-formats.md](./output-formats.md).

Under `decision:functional-parity` this spec fixes names, arity, value types, accepted and refused combinations, and the
class of each refusal. It does not fix the wording or layout of help, usage, or error text; those are the argument
parser's and the binary's own.

## Interfaces

### Identity and root

```text
# product:component-help-version, decision:parallel-workspaces/component-runtime/program-name
binary name            = winter-workspace
program name in output = "winter-workspace", fixed; not derived from argv[0]
winter-workspace --version
  stdout: one line carrying the program name and the package version   exit 0
winter-workspace --help
  stdout: root help                                                    exit 0
winter-workspace            (no arguments)
  stderr: root help                                                    exit 2
```

`--version`, `--help`, a bare group, and every usage error work outside a workspace and with a malformed
`.winter/config.toml`: the workspace is located and configuration loaded only when a command first needs it (reference:
`cli_context.py:13-31`, `cli.py:429`).

| Root option                           | Type, default | Visible | Behaviour                                                                                                                                                                                  |
| ------------------------------------- | ------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--version`                           | flag          | yes     | Prints the version line and exits 0 before any other processing. No `-V`.                                                                                                                  |
| `--service-orchestrator PATH_OR_NAME` | string, unset | yes     | Provider override for this invocation; falls back to `WINTER_SERVICE_ORCHESTRATOR`, then to configuration. When effective, `service` commands write one notice naming the value on stderr. |
| `-v`, `--verbose`                     | flag, off     | yes     | DEBUG logging on stderr; wins over `WINTER_LOG_LEVEL`.                                                                                                                                     |
| `--help`                              | flag          | yes     | Prints help to stdout and exits 0. Present on every node. No `-h`.                                                                                                                         |
| `--source-override TEXT`              | string, unset | hidden  | Consumed only by the dashboard title.                                                                                                                                                      |

Root options are accepted only before the first command token. There is no global `--json`, `--quiet`, `--color`,
`--cwd`, or workspace-root option, and no option reads its value from an environment variable through the parser.

### Tree

50 nodes: 5 groups (root, `ext`, `repo`, `service`, `ws`), 45 leaf registrations, 43 distinct leaf commands, 2 aliases,
3 hidden commands, 32 nodes with `--json`.

```text
winter-workspace
├── agents                         [--json]
├── capabilities                   [--json]
├── clean PATTERNS...              [--json]
├── dashboard
├── doctor                         [--json]
├── env SCOPE
├── ext
│   ├── new NAME                   # presence open: decision:agent-ready-operation/executable-extensions/scaffold-command
│   └── verify EXTENSIONS...       [--json]
├── graph                          [--json]
├── lint [SCOPES]...               [--json]
├── provision PATTERNS...          [--json]
├── repo
│   ├── add URL                    [--json]
│   ├── list                       [--json]
│   └── remove TARGET              [--json]
├── service
│   ├── catalog                    hidden; always JSON
│   ├── describe                   hidden
│   ├── down PATTERNS...
│   ├── ext-services               hidden; always JSON
│   ├── logs PATTERNS...
│   ├── restart PATTERNS...
│   ├── start PATTERNS...          alias of up
│   ├── status [PATTERNS]...       [--json]
│   ├── stop PATTERNS...           alias of down
│   └── up PATTERNS...
├── space KIND
└── ws
    ├── checkout ENV FEATURE_BRANCH        [--json]
    ├── clean PATTERNS...                  [--json]
    ├── connect PATTERNS... FEATURE_BRANCH [--json]
    ├── destroy PATTERNS...                [--json]
    ├── diff [PATTERNS]...                 [--json]
    ├── disconnect PATTERNS...             [--json]
    ├── fetch [PATTERNS]...                [--json]
    ├── fingerprint                        [--json]
    ├── index NAME                         [--json]
    ├── init [TARGET]                      [--json]
    ├── list                               [--json]
    ├── merge SOURCE_REF [PATTERNS]...     [--json]
    ├── prune                              [--json]
    ├── pull [PATTERNS]...                 [--json]
    ├── push [PATTERNS]...                 [--json]
    ├── reset PATTERNS... REF              [--json]
    ├── restack ENV... BASE                [--json]
    ├── status [PATTERNS]...               [--json]
    ├── update [REPOS]...                  [--json]
    └── worktrees                          [--json]
```

### Parsing rules

```text
# reference: cli.py:165-261; modules/service/command.py:120-149
every group: bare invocation prints that group's help on stderr, exit 2
every node:  --help prints that node's help on stdout, exit 0
hidden:      hidden commands and options are omitted from help and still dispatch
aliases:     `service start` / `service stop` are the `up` / `down` commands under a second name: same options,
             same behaviour, same exit codes; the provider action word stays `up` / `down`
options:     may appear before, between, or after a command's positionals; `--` ends option parsing
names:       no prefix matching for commands or long options; no passthrough of unknown options
variadic:    PATTERNS... requires one or more tokens; [PATTERNS]... accepts zero or more
usage error: unknown command or option, missing required positional or option value, value of the wrong type —
             exit 2, nothing started
```

Three commands declare one variadic positional and split the last token off themselves.

| Command      | Tokens                       | Last token       | Zero tokens         | Exactly one token     | Reference                          |
| ------------ | ---------------------------- | ---------------- | ------------------- | --------------------- | ---------------------------------- |
| `ws connect` | `PATTERNS... FEATURE_BRANCH` | `FEATURE_BRANCH` | usage error, exit 2 | command error, exit 1 | `modules/workspace/command.py:359` |
| `ws reset`   | `PATTERNS... REF`            | `REF`            | usage error, exit 2 | command error, exit 1 | `modules/workspace/command.py:544` |
| `ws restack` | `ENV... BASE`                | `BASE`           | usage error, exit 2 | command error, exit 1 | `modules/workspace/command.py:704` |

### Commands

`--json` column: **A** one document, **B** one document per line, **C** NDJSON event stream, as defined in
[output-formats.md](./output-formats.md). Every flag defaults to off. `*` marks a repeatable option. The Purpose column
describes the command; it is not the help summary, whose wording is free.

#### Top level

| Command        | Arguments     | Options                                                                                                                                  | `--json` | Purpose                                                                                                |
| -------------- | ------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | -------- | ------------------------------------------------------------------------------------------------------ |
| `agents`       | —             | `--json`                                                                                                                                 | B        | Resolved model and effort for every installed agent × harness, with the tables each was resolved from. |
| `capabilities` | —             | `--json`                                                                                                                                 | B        | Every capability slot: bound provider, other candidates, whether each entrypoint resolves.             |
| `clean`        | `PATTERNS...` | `--stage dependency\|resource\|data`; `--name SCOPE.NAME`; `--no-service-check`; `--dry-run`; `--json`                                   | C        | Run each matched provision handler's declared `clean` command across the matched feature envs.         |
| `dashboard`    | —             | —                                                                                                                                        | —        | Launch the terminal dashboard.                                                                         |
| `doctor`       | —             | `--json`                                                                                                                                 | C        | Preflight checks for the workspace and every installed extension.                                      |
| `env`          | `SCOPE`       | `--resolve` (run command entries and print real values instead of masks)                                                                 | —        | Print the runtime environment variables for SCOPE as sourceable `export` lines.                        |
| `graph`        | —             | `--json`                                                                                                                                 | B        | The module dependency graph declared by `winter-ext.toml` `requires`.                                  |
| `lint`         | `[SCOPES]...` | `--all` (every feature env's project repos); `--changed` (dirty / un-pushed files only); `--json`; `--show-ignored`                      | C        | Convention lint checks over the project repos of a feature env.                                        |
| `provision`    | `PATTERNS...` | `--stage dependency\|resource\|data`; `--reset`; `--destroy`; `--seed`; `--name SCOPE.NAME`; `--no-service-check`; `--dry-run`; `--json` | C        | Provision the matched feature envs: install dependencies, create resources, load data.                 |
| `space`        | `KIND`        | —                                                                                                                                        | —        | Print the artifact directory for KIND; resolves a path and creates nothing.                            |

#### `ext`

| Command      | Arguments       | Options                                                 | `--json` | Purpose                                                                            |
| ------------ | --------------- | ------------------------------------------------------- | -------- | ---------------------------------------------------------------------------------- |
| `ext new`    | `NAME`          | `--capability TEXT` (required); `--dir TEXT`; `--force` | —        | Scaffold a new extension implementing a capability slot.                           |
| `ext verify` | `EXTENSIONS...` | `--json`                                                | B        | Verify each extension against the service capability spec; one result per operand. |

Each `ext verify` operand is an installed extension's name or a local path to an extension directory; there is no slot
operand (`modules/ext/command.py:32-34`). It exits 1 when any operand fails verification.

#### `repo`

| Command       | Arguments | Options                                                                                                                                                                                                                       | `--json` | Purpose                                               |
| ------------- | --------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------- | ----------------------------------------------------- |
| `repo add`    | `URL`     | `--standalone`; `--name TEXT`; `--main-branch TEXT`; `--ref TEXT`; `--git-exclude TEXT`\*; `--cmd TEXT`\*; `--pinned` (project only); `--path TEXT` (standalone only); `--prefix TEXT` (standalone only); `--local`; `--json` | A        | Add a repository to the workspace configuration.      |
| `repo list`   | —         | `--json`                                                                                                                                                                                                                      | A        | List all repositories.                                |
| `repo remove` | `TARGET`  | `--local`; `--json`                                                                                                                                                                                                           | A        | Remove a repository from the workspace configuration. |

#### `service`

| Command                | Arguments       | Options                                                                                                                                                                         | `--json`    | Purpose                                                                    |
| ---------------------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------- | -------------------------------------------------------------------------- |
| `service up`           | `PATTERNS...`   | `--wait`; `--timeout SECONDS` (float, default `120.0`)                                                                                                                          | —           | Start services in the matched scopes.                                      |
| `service start`        | `PATTERNS...`   | as `up`                                                                                                                                                                         | —           | Alias of `up`.                                                             |
| `service down`         | `PATTERNS...`   | —                                                                                                                                                                               | —           | Stop services in the matched scopes.                                       |
| `service stop`         | `PATTERNS...`   | —                                                                                                                                                                               | —           | Alias of `down`.                                                           |
| `service restart`      | `PATTERNS...`   | —                                                                                                                                                                               | —           | Restart matched services without stopping unmatched ones.                  |
| `service status`       | `[PATTERNS]...` | `--json`                                                                                                                                                                        | A           | Report the state of matched services; defaults to every scope and service. |
| `service logs`         | `PATTERNS...`   | `-f`, `--follow`; `-n`, `--tail N` (text, default `"200"`; positive integer or `all`); `--since DUR\|TS` (default `""`); `--until DUR\|TS` (default `""`); `-t`, `--timestamps` | —           | Relay the log backlog or live stream of matched services.                  |
| `service catalog`      | —               | —                                                                                                                                                                               | always B    | Hidden. Scope-qualified service names across providers.                    |
| `service ext-services` | —               | —                                                                                                                                                                               | always B    | Hidden. The aggregated extension service definitions.                      |
| `service describe`     | —               | —                                                                                                                                                                               | passthrough | Hidden. Forwards the provider's `describe` output.                         |

`--since` / `--until` accept a duration `<N>(s|m|h|d)` or an RFC 3339 timestamp. An invalid `--tail`, `--since`, or
`--until` is a usage error, exit 2.

#### `ws`

| Command          | Arguments                    | Options                                                                                                                  | `--json`                    | Purpose                                                                            |
| ---------------- | ---------------------------- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------- | ---------------------------------------------------------------------------------- |
| `ws checkout`    | `ENV` `FEATURE_BRANCH`       | `--force`; `--new`; `--json`                                                                                             | A                           | Adopt a remote feature branch into ENV, all-or-nothing.                            |
| `ws clean`       | `PATTERNS...`                | `--force`; `--dry-run`; `--json`                                                                                         | C                           | Remove untracked files from matched worktrees.                                     |
| `ws connect`     | `PATTERNS... FEATURE_BRANCH` | `--json`                                                                                                                 | A                           | Connect matched worktrees to a remote feature branch.                              |
| `ws destroy`     | `PATTERNS...`                | `--force`; `--strict`; `--dry-run`; `--no-provision-teardown`; `--json`                                                  | C                           | Tear down the matched feature envs.                                                |
| `ws diff`        | `[PATTERNS]...`              | `--staged`; `--branch`; `--no-headers`; `--json`                                                                         | A                           | Unified diff across matched repos.                                                 |
| `ws disconnect`  | `PATTERNS...`                | `--json`                                                                                                                 | A                           | Disconnect matched worktrees from their feature branch.                            |
| `ws fetch`       | `[PATTERNS]...`              | `--standalone`; `--all`; `--json`                                                                                        | C                           | Fetch refs from `origin` for matched project worktrees.                            |
| `ws fingerprint` | —                            | `--json`                                                                                                                 | A                           | Print one digest identifying the workspace definition.                             |
| `ws index`       | `NAME`                       | `--json`                                                                                                                 | A                           | Print the port-offset index for a feature env name.                                |
| `ws init`        | `[TARGET]`                   | `--all`; `--json`                                                                                                        | C                           | Reconcile source checkouts and standalone repos, one feature env, or everything.   |
| `ws list`        | —                            | `--json`                                                                                                                 | A (array)                   | List all feature envs.                                                             |
| `ws merge`       | `SOURCE_REF` `[PATTERNS]...` | `--standalone`; `--all`; `--ff-only`; `--merge`; `--no-ff`; `--autostash`; `--exclude-pinned`; `--only-pinned`; `--json` | C                           | Merge SOURCE_REF into matched project worktrees.                                   |
| `ws prune`       | —                            | `--dry-run`; `--force`; `--json`                                                                                         | A, C events may precede     | Remove disk state for repos no longer in the configuration.                        |
| `ws pull`        | `[PATTERNS]...`              | `--standalone`; `--all`; `--ff-only`; `--merge`; `--rebase`; `--autostash`; `--json`                                     | C                           | Fetch and integrate matched project worktrees.                                     |
| `ws push`        | `[PATTERNS]...`              | `--standalone`; `--all`; `--include-pinned`; `--only-pinned`; `--json`                                                   | A                           | Push matched project worktrees to their tracked branch.                            |
| `ws reset`       | `PATTERNS... REF`            | `--soft`; `--mixed` (the default mode); `--hard`; `--force`; `--dry-run`; `--json`                                       | C                           | Move matched worktree branches to REF, all-or-nothing.                             |
| `ws restack`     | `ENV... BASE`                | `--cut TEXT`; `--dry-run`; `--json`                                                                                      | A                           | Restack a chain of feature envs onto BASE.                                         |
| `ws status`      | `[PATTERNS]...`              | `--json`; `--fetch`                                                                                                      | A; with `--fetch`, C then A | Show workspace status, filtered by PATTERNS.                                       |
| `ws update`      | `[REPOS]...`                 | `--autostash`; `--freeze`; `--force`; `--json`                                                                           | C                           | Re-resolve `ref` pins for standalone repos and rewrite the lock.                   |
| `ws worktrees`   | —                            | `--json`; `--status`                                                                                                     | A (array)                   | List feature-env `<env>/<repo>` worktrees; `--status` adds ahead / behind / dirty. |

Target selection, per-command defaults, and the meaning of each option belong to the spec of the epic that owns the
command; this table fixes names, arity, and types. Confirmation rules are in
[results-and-exit-codes.md](./results-and-exit-codes.md).

### Combination rules

Mutual exclusion, required combinations, and operand shape are checked by the command, after parsing succeeds and before
anything is read from git or written. A violated rule is a command error (exit 1) unless its class says usage (exit 2);
the error names the flags or the operand at fault. Reference: `modules/workspace/command.py`,
`modules/workspace/handlers/`, `modules/provision/command.py`, `modules/ext/command.py`.

Pattern shapes:

| Shape              | Used by                                                                                          | Refused                              |
| ------------------ | ------------------------------------------------------------------------------------------------ | ------------------------------------ |
| `<env>/<repo>`     | `ws connect`, `reset`, `fetch`, `pull`, `merge`, `push`, `clean`, `diff`, `status`, `disconnect` | an empty pattern; more than one `/`  |
| env-level          | `ws destroy`, `provision`, `clean`                                                               | an empty pattern; any `/`            |
| bare name          | `lint` SCOPES, `ws update` REPOS                                                                 | an empty name; any `/`               |
| literal chain name | `ws restack` ENV                                                                                 | empty; any of `*`, `?`, `[`; any `/` |

Rules per command, in the order they are checked; the first violated rule is the one reported.

| Command                                            | Checks, in order                                                                                                                                                                                                                                                                                                                        |
| -------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `ws connect`                                       | exactly one token; empty FEATURE_BRANCH; pattern shape                                                                                                                                                                                                                                                                                  |
| `ws reset`                                         | exactly one token; empty REF; pattern shape; two or more of `--soft`, `--mixed`, `--hard`                                                                                                                                                                                                                                               |
| `ws restack`                                       | exactly one token; chain-name shape of each ENV; empty BASE                                                                                                                                                                                                                                                                             |
| `ws fetch`                                         | PATTERNS with `--standalone`; `--standalone` with `--all`; pattern shape                                                                                                                                                                                                                                                                |
| `ws pull`                                          | PATTERNS with `--standalone`; `--standalone` with `--all`; two or more of `--ff-only`, `--merge`, `--rebase`; pattern shape                                                                                                                                                                                                             |
| `ws merge`                                         | PATTERNS with `--standalone`; `--exclude-pinned` or `--only-pinned` with `--standalone`; `--standalone` with `--all`; no PATTERN unless `--standalone` is given (so `ws merge REF` and `ws merge REF --all` are both refused); two or more of `--ff-only`, `--merge`, `--no-ff`; `--exclude-pinned` with `--only-pinned`; pattern shape |
| `ws push`                                          | PATTERNS with `--standalone`; `--include-pinned` or `--only-pinned` with `--standalone`; `--standalone` with `--all`; `--include-pinned` with `--only-pinned`; pattern shape                                                                                                                                                            |
| `ws diff`                                          | `--staged` with `--branch`; pattern shape                                                                                                                                                                                                                                                                                               |
| `ws clean`                                         | pattern shape; `--json` without `--force` or `--dry-run`                                                                                                                                                                                                                                                                                |
| `ws update`                                        | `--force` without `--freeze` (usage); `--autostash` with `--freeze` (usage); bare-name shape (command)                                                                                                                                                                                                                                  |
| `ws init`                                          | TARGET with `--all`; TARGET equal to `workspace`, the reserved service scope                                                                                                                                                                                                                                                            |
| `ws status`, `ws disconnect`, `ws destroy`, `lint` | pattern shape only                                                                                                                                                                                                                                                                                                                      |
| `provision`                                        | `--reset` with `--destroy`; `--seed` without `--stage resource`; `--seed` with `--reset` or `--destroy`; `--seed` with `--name`; `--reset` or `--destroy` with neither `--stage` nor `--name`; pattern shape                                                                                                                            |
| `repo add`                                         | URL shape; with `--standalone`: `--pinned`; without `--standalone`: `--path`, then `--prefix`, then `--ref`; `--path` absolute or with a `..` segment; URL already declared for a repository of that kind, then name already declared; standalone resolved path already declared                                                        |
| `repo remove`                                      | TARGET without a `/`; then, TARGET split at its first `/` into type and name: type neither `project` nor `standalone`; no repository of that type with that name (so `project/a/b` is "not found" for the name `a/b`)                                                                                                                   |
| `ext new`                                          | unknown capability slot (usage)                                                                                                                                                                                                                                                                                                         |

```text
# repo add URL shape — reference: core/internal/click_cli_input_validation_service.py:21-27
accepted:  (http|https|ssh|git)://<host>[:<port>][/<path>][.git][/]
           <user>@<host>:<path>[.git]
host, user: letters, digits, "_", ".", "+", "-"      path: those plus "/" and "~"
anything else is refused before any other check
```

### Process inputs

Environment variables the binary reads from its own environment. No variable selects the workspace root, colour, or JSON
mode.

| Variable                                                                            | Use                                                                                                                                                                                                                                                               | Reference                                                                   |
| ----------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------- |
| `WINTER_LOG_LEVEL`                                                                  | Log level name, case-insensitive; an unknown or empty name is ignored silently.                                                                                                                                                                                   | `cli.py:278-283,332`                                                        |
| `WINTER_SERVICE_ORCHESTRATOR`                                                       | Provider override; the root flag wins.                                                                                                                                                                                                                            | `cli.py:336`                                                                |
| `WINTER_INVOCATION_CWD`                                                             | The caller's directory, used by `ext new` to resolve the output directory and by `lint` to detect the current env and locate the repo for `--changed`. Unset or empty: the process working directory.                                                             | `modules/ext/command.py:100`, `modules/lint/command.py:76`                  |
| `WINTER_LAUNCH_TIME`                                                                | Start instant of the command span: epoch seconds with a `.` or `,` fraction. Read once at start-up and removed from the environment, so no child inherits it. Unset, unparseable, non-positive, or in the future: the span starts when the command is dispatched. | `cli.py:366-372`, `core/tracing.py:62-67`                                   |
| `WINTER_GIT_TIMEOUT_S`                                                              | Per-call remote git timeout, float seconds, default `40`; invalid or ≤ 0 warns and uses the default.                                                                                                                                                              | `modules/workspace/internal/git_ops_service.py:39,135-149`                  |
| `WINTER_NESTED_CHAIN`                                                               | Path-separator-joined workspace roots of the enclosing nested calls; recursion guard.                                                                                                                                                                             | `modules/workspace/nested_env.py:12,28,66`                                  |
| `WINTER_OTEL_EXPORTER_OTLP_ENDPOINT`, `…_HEADERS`                                   | Tracing switch, endpoint, and export headers.                                                                                                                                                                                                                     | `cli.py:374-375`                                                            |
| `OTEL_SDK_DISABLED`, `TRACEPARENT`, `OTEL_SERVICE_NAME`, `OTEL_RESOURCE_ATTRIBUTES` | Tracing kill switch, parent context, span resource.                                                                                                                                                                                                               | `cli.py:376`, `core/internal/otel_command_tracer.py:85-103`                 |
| `GIT_SSH_COMMAND`                                                                   | Set to `ssh -o ConnectTimeout=10 -o ServerAliveInterval=30 -o ServerAliveCountMax=3` when unset, before argument parsing, for every invocation.                                                                                                                   | `core/ssh_keepalives.py:12-25`                                              |
| `RUST_BACKTRACE`, `RUST_LIB_BACKTRACE`                                              | Local development only: a backtrace follows an internal-failure diagnostic. No reference equivalent.                                                                                                                                                              | `decision:parallel-workspaces/component-runtime/internal-failure-rendering` |

`WINTER_INVOCATION_CWD` and `WINTER_LAUNCH_TIME` are honoured when a caller sets them. In the reference setup only the
bash launcher does; nothing in this binary sets either, so a direct invocation takes the stated fallback.

### Deliberately absent

| Reference behaviour                                                                | Why absent                                                                                                                              |
| ---------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| `winter init` (create an empty `.winter/config.toml`, then `ws init`)              | Launcher-only; `decision:parallel-workspaces/component-runtime/launcher-only-surface`.                                                  |
| `--winter=PATH` source selection                                                   | Launcher-only; same decision.                                                                                                           |
| Virtual-environment freshness check, dependency sync, `VIRTUAL_ENV` / `PATH` setup | Launcher-only; same decision.                                                                                                           |
| Launcher root rule requiring `tools/winter-cli/` beside `.winter/config.toml`      | Launcher-only; root discovery is [configuration-model.md](../../workspace-configuration/spec/configuration-model.md).                   |
| Bytecode cache redirect (`XDG_CACHE_HOME`)                                         | Python-runtime behaviour with no observable equivalent.                                                                                 |
| `doctor` python-version probe                                                      | Dropped; registered when the diagnostics spec is written.                                                                               |
| Commands contributed by in-process plugins                                         | Never reached the reference tree; plugin loading is dropped by `decision:agent-ready-operation/workspace-dashboard/contribution-scope`. |
| A `context` command                                                                | New capability, designed in `epic:agent-ready-operation/context-navigation`; not part of this surface.                                  |
| `component.toml`, `.winterrc`, handoff metadata                                    | `decision:rust-replacement`: direct invocation is the only entry point.                                                                 |

### Registered deviations

Every place the binary's function differs from the reference tool on purpose (`decision:rough-parity`). A difference of
function not listed here or under Deliberately absent is a defect. Differences of wording, layout, and language-native
behaviour are not listed: they fall under `decision:functional-parity`.

| Deviation                                                                     | Reference behaviour                                                                                  | Binary                                                                   | Decision                                                                       |
| ----------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| Launcher-only commands, options, and files                                    | present                                                                                              | absent; rows above                                                       | `decision:parallel-workspaces/component-runtime/launcher-only-surface`         |
| Failed git call before a repo error                                           | failure text written twice on stderr                                                                 | written once; a log record as well only under `-v` or `WINTER_LOG_LEVEL` | `decision:parallel-workspaces/component-runtime/git-failure-rendering`         |
| Non-list value for `git_excludes` or `cmd`; non-string `main_branch` or `ref` | a string is read as its characters and a table as its keys; other wrong types fail unhandled, exit 1 | configuration error naming the key, exit 1                               | `decision:parallel-workspaces/workspace-configuration/element-type-validation` |
| In-process plugins                                                            | `plugin.py` files and plugin directories are imported                                                | none is loaded; the tmux service badge is native                         | `decision:agent-ready-operation/workspace-dashboard/contribution-scope`        |
| `repo remove` of a block with comment lines directly above it or the next     | keeps the lines above the removed block; removes the lines above the next block with it              | removes the lines above the removed block; keeps those above the next    | `decision:parallel-workspaces/repository-catalog/removal-comment-extent`       |

## Open questions

- Whether a repeated flag (`--json --json`) or a repeated single-value option is accepted with the last value winning,
  as the reference parser does, is not recorded in any observation. Engineering answers it when the parser scenarios are
  recorded.
- Whether `ext new` stays in the tree is `decision:agent-ready-operation/executable-extensions/scaffold-command`; the
  product owner answers when `epic:agent-ready-operation/executable-extensions` is refined. Until then its rows stand as
  the reference tool's.
