# Reconciliation

Bare `ws init`: making each declared source checkout and standalone exist, verified, and set up, then refreshing the
workspace's managed content. Its argument and options are the `ws init` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands, and
its refusals that file's `ws init` row of Combination rules. Shape C and its framing are those of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md); `RepoError`,
`RepoMessage`, and `Termination` are those of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
Domain. Every git call goes through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md) with the
child environment it states; the pin step is `materialize` of
[standalone-pins.md](../../reproducible-inputs/spec/standalone-pins.md), Flows, Materialize a pin. The managed blocks
are those of [managed-content.md](./managed-content.md). Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`. Message wording is the binary's own
(`decision:functional-parity`).

## System

```text
commands/ws/init.rs ── reconcile(ports, env, diagnostics, start, sink) ──► bootstrap::reconcile
                                                                            ├─ projects target ──┐
                                                                            ├─ standalone target ┼─► per repository, at most
                                                                            │                    │   4 at once: checkout::run
                                                                            ├─ workspace_exclude::finalize
                                                                            └─ agents_md::refresh
TARGET and --all reach epic:parallel-workspaces/environment-lifecycle, which adds them to this command file and calls
`reconcile` for the workspace part of --all. A build holds what its files register (command-surface.md: the tree lists
what the finished binary has): a build without that epic registers `ws init [--json]`, and TARGET or --all there is
the usage error any unregistered argument or option is, exit 2. The finished binary keeps the reference scopes,
decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope
```

Steps that other epics add to a repository or to the run, each after the step named:

| Step                                                         | After                    | Owner                                            |
| ------------------------------------------------------------ | ------------------------ | ------------------------------------------------ |
| Extension projection, retraction, `AGENTS.winter.md`, hooks  | the standalone's pin     | `epic:agent-ready-operation/extension-management` |
| Extension lines of an eligible exclude block (`extra`)       | `finalize`'s eligible    | `epic:agent-ready-operation/extension-management` |
| Workspace skill projection into the vendor skill directories | the projects fan-out | `epic:agent-ready-operation/harness-projection`  |
| Nested workspace reconciliation                              | the repository's `cmd`   | `epic:parallel-workspaces/nested-workspaces`     |

## Domain

```rust
// crates/core/src/bootstrap.rs — engineering:typed-outcomes, engineering:bootstrap-effect-accounting
pub enum InitEvent {
    TargetStarted { target: String },                          // "projects/" | "standalone/"
    TargetCompleted { target: String, success: bool },
    RepoAction { repo: String, location: String, action: Action, detail: String },
    RepoError { repo: String, error: String },                 // the rendered RepoError, or the I/O error's text
    CmdStarted { repo: String, command: String },
    CmdOutputLine { repo: String, line: String },
    CmdCompleted { repo: String, command: String, returncode: i32 },
}
pub enum Action {                                               // the `action` value
    Cloned,                     // "cloned"                      detail ""
    Exists,                     // "exists"                      detail ""
    ExcludesUpdated,            // "excludes_updated"            detail: the appended entries joined by ", "
    Pinned,                     // "pinned"                      detail: "<kind> <commit, first 8>"
    WorkspaceExcludesUpdated,   // "workspace_excludes_updated"  detail below
    AgentsMdUpdated,            // "agents_md_updated"           detail ""; registered deviation, creation-ownership
}
pub enum InitError { Config(Box<ConfigError>), Internal(Box<InternalError>) }   // the run ends before any target
pub type InitSink<'s> = dyn FnMut(&InitEvent, &mut Diagnostics<'_>) -> Result<(), Termination> + 's;
pub enum RunEnd { Completed { success: bool }, Stopped(Termination) }          // Stopped: a failed stdout write
pub fn reconcile(ports: &Ports<'_>, env: &ProcessEnv, diagnostics: &mut Diagnostics<'_>, start: &Path,
    sink: &mut InitSink<'_>) -> Result<RunEnd, InitError>;

// crates/core/src/bootstrap/checkout.rs — one repository's steps, in this order, each after the one before succeeds
pub(crate) struct Checkout<'r> {
    pub name: &'r str, pub url: Option<&'r str>, pub location: PathBuf,
    pub git_excludes: Vec<String>,                              // the workspace's git_excludes, then the repository's
    pub cmd: &'r [String], pub nested: bool,                     // nested: a project with `nested` declared
    pub standalone: Option<&'r StandaloneRepository>,           // Some for a standalone: the pin step runs
}
```

```rust
// crates/core/src/termination.rs — RepoMessage gains, each carried by a RepoError with the git call when there was one
NoUrl { name },                         // "no `url` declared in config; cannot clone"
NotARepository { name, path },          // the location exists and is not the top of a git work tree
OriginMismatch { name, path, declared, found: Option<String> },   // found None: no remote.origin.url
CommandFailed { name, command, returncode },                      // "`<command>` exited with code <returncode>"
CommandNotStarted { name, command, cause },                       // "`<command>` — <cause>"
UnsafeLocation { name, path, reason },                            // clone-confinement; nothing is created
// GitOperation gains Clone, Config, ExcludeLocate, Toplevel, OriginUrl, LocalOnly, StashList (the last two: prune.md)
```

## Interfaces

### Git operations

```text
# crates/core/src/git.rs, beside those of standalone-pins.md; the operation span is named as the git subcommand,
# attribute winter.repo, as standalone-pins.md, Operation spans
clone(url, dest)          clone -- <url> <dest>         cwd the workspace root; no depth, branch, or recursion: the
                                                        remote's default HEAD; no timeout and no retry, as the
                                                        reference's clone_from (gitpython_repository.py:42-51)
toplevel(path)            rev-parse --show-toplevel     non-zero exit -> None
origin_url(path)          config --get remote.origin.url   exit 1 -> None
set_config(path, k, v)    config <k> <v>                the repository's own config (.git/config)
```

### Running `ws init`

```text
# reference: modules/workspace/handlers/init_handler.py:31-50, modules/workspace/init_service.py:116-221,348-358
order
  1. root discovery, then the load (configuration-model.md); a failure is InitError::Config, exit 1, nothing written
  2. projects target
  3. standalone target, or finalize alone when it is skipped
  4. agents_md::refresh
  exit 1 when any target completed with success false, any RepoError event was emitted, or a step of 3 or 4 failed;
  else 0. Every step runs whatever an earlier one reported.

projects target                                   # reference: init_service.py:116-137
  TargetStarted "projects/"
  create_dir_all(<root>/projects); an io::Error -> RepoError event repo "winter", target success false
  write_block("winter-dir/projects", ["/projects/"])          -> Some(file): RepoAction repo "winter", location file,
  write_block("winter-workspace/artifacts", ["/.winter/logs/"])   WorkspaceExcludesUpdated, detail the block's one line
                                                                a GitFailure or io::Error -> RepoError repo "winter",
                                                                "workspace exclude — <cause>", success false
  fan-out over the projects, in merged order: checkout::run with location <root>/projects/<name>
  write_block("winter-workspace/workspace-skills", [".claude/skills/<p>-*", ".claude/skills/<p>", ".codex/skills/<p>-*",
    ".codex/skills/<p>", ".opencode/skill/<p>-*", ".opencode/skill/<p>"]), p the workspace `prefix`, written whether
    or not a skills directory exists, and reported by no event; a GitFailure or io::Error -> RepoError repo
    "workspace", "workspace skills — <cause>", success false (reference: init_service.py:133,
    workspace_skill_service.py:101-145). The projection the block covers is harness-projection's
  TargetCompleted "projects/", success

standalone target                                 # reference: init_service.py:139-221
  skipped when the workspace declares no standalone and no project is an extension, context-only, or opted out; then
  only finalize runs, with no eligible, and no target event is emitted (reference: init_service.py:141-150,
  repository_factory.py:135-287). Over the projects not also declared as standalones:
    extension      `extension` not false and <root>/projects/<name>/winter-ext.toml a file
    context-only   `extension` not false, `load` not "none", `load` or `entry` declared, and no winter-ext.toml file
    opted out      `extension = false`
  # product:reconcile-repositories, "omission is explained": the `ws/init` topic and the command's help state what
  # bare ws init leaves out, feature environments and a standalone target with nothing to reconcile; the output adds
  # no line for it, as the reference adds none
  TargetStarted "standalone/"
  fan-out over the standalones, in merged order: checkout::run with location standalone_location(root, name, path)
  finalize(mode, eligible, keep: []): eligible := each standalone whose location exists, in merged order, then each
    project that is an extension, as above, in merged order, as { name, relative: the location relative to the root
    with "/" separators, extra: [] }; a location outside the root is not eligible (reference:
    init_service.py:177-206); its block is the path line alone until extension-management adds `extra`
    Updated -> RepoAction repo "winter-extensions", location file, WorkspaceExcludesUpdated, detail the names joined
               by ", ", or "cleared" when none
    an error -> RepoError repo "winter-extensions", success false
  TargetCompleted "standalone/", success

agents_md::refresh                                # registered deviation: decision:parallel-workspaces/workspace-bootstrap/creation-ownership
  Updated  -> RepoAction repo "workspace", location <root>/AGENTS.md, AgentsMdUpdated
  Conflict -> RepoError repo "workspace", naming the file and the conflict; the file is not written
  Created, Unchanged, Absent -> nothing
```

### One repository

```text
# reference: modules/workspace/init_service.py:398-453,485-561,844-936
# product:reconcile-repositories, product:recorded-input-materialization, product:bootstrap-recovery
checkout::run(c) -> Result<(), RepoError or io::Error>; the first Err is the repository's RepoError event, and every
                    later step of that repository is not attempted. The event's text names the step that failed and
                    the steps not attempted, in this order: location, identity, excludes, cmd, pin; with the events
                    before it, a reader has each step completed, failed, or not attempted (product:bootstrap-recovery)
  1. location
     absent: c.url None -> NoUrl
             a standalone: confined(location) fails -> UnsafeLocation   # decision:parallel-workspaces/workspace-bootstrap/clone-confinement
             create_dir_all(parent); clone(url, location) -> RepoAction Cloned
     present:                                          # decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope;
       toplevel(location) != resolve(location) -> NotARepository        # a registered deviation: the reference
       c.url Some(u) and origin_url(location) != Some(u) -> OriginMismatch   # reports exists without any check
       -> RepoAction Exists       # HEAD is not checked: a clone of an empty remote has none and is a valid clone
       urls compare as exact strings: `…/a` and `…/a.git` differ, as in repo add's duplicate check
  2. identity     the workspace git identity, when one exists: set_config user.name, then user.email; written every run
  3. excludes     entries := c.git_excludes, each trimmed, empty ones dropped
                  none -> skip
                  file := exclude_path(location); None -> RepoError "could not locate .git/info/exclude at <location>"
                  update(file): new := entries not already a trimmed line of the file, in order, without repeats;
                    none -> no write; else append ["", "# winter-managed", new...] where the "" is dropped when the
                    file is empty, and the result ends with "\n"
                  -> RepoAction ExcludesUpdated, location c.location, detail new joined by ", "
                  # append-only: an entry later removed from the configuration stays in the file, as in the reference
  4. cmd          Commands below
  5. pin          standalones only: materialize(ports, root, repo)
                  Some((kind, commit)) -> RepoAction Pinned "<branch|tag|commit> <commit[..8]>"; None -> nothing
                  Err -> the RepoError event, its rendered PinError
                  # PinError::Config, a lock that is unreadable, malformed, or of an unsupported version, is how
                  # product:recorded-input-materialization's "is a configuration error" reads under fan-out: the
                  # standalone fails with the ConfigError's text, the run exits 1, no record is written, and every
                  # other repository and its record go on, as the reference's per-repository reconcile does
confined(location)                                # registered deviation; the reference clones wherever the path leads
  l := resolve(location); r := resolve(root); refused, with the reason, when
    l is r, or l is not under r                                                   "outside the workspace"
    l is under r/.winter or r/projects                                            "reserved directory"
    l is, contains, or lies under resolve of another declared repository's location: a project's
      <root>/projects/<name>, another standalone's standalone_location            "overlaps <name>"
  resolve is that of repository-addition.md, Duplicates: a missing component is kept as text
  repository-addition.md = ../../../../delivered/parallel-workspaces/repository-catalog/spec/repository-addition.md
exclude_path(location)
  <location>/.git a directory -> <location>/.git/info/exclude
  <location>/.git a file starting "gitdir:" -> d := the rest, trimmed, resolved against location when relative;
    d/commondir a file -> resolve(d / its trimmed content)/info/exclude; else d/info/exclude
  otherwise None
```

### Commands

```text
# engineering:process-execution: `cmd` is a configuration-declared command line, so it runs through the shell
# reference: modules/workspace/init_service.py:42-53,912-936, modules/workspace/nested_env.py:31-55
program       /bin/sh -c <command>         Program::Shell of programs.rs
cwd           the repository's location
stdin         inherited from the binary, as the reference's Popen leaves it: Call gains `stdin: Stdin`, Null for every
              git call, Inherit here. OsPrograms::new gains `inherit_stdin: bool`, which process.rs's `run` takes
              from its caller: main.rs passes true, InProcess and every other test false, where Inherit reads as
              Null; so a component test's child reads empty input and main.rs stays the one file that chooses the
              process streams. A byte the
              binary's own stdin reader has buffered is not seen by the child; no command of this epic reads stdin
              before its children run. context/architecture/structure.md records the exception
stdout        one pipe, which the child's stderr shares; read as it arrives and split into lines at "\n", each line
              without its "\n" and decoded as UTF-8 with invalid bytes replaced; a last line without "\n" is a line
environment   the snapshot's names and values, as the reference's os.environ.copy() holds them: WINTER_LAUNCH_TIME
              left out and GIT_SSH_COMMAND defaulted, as child-processes.md gives for git, but LANGUAGE and LC_ALL
              not set to C, so the user's locale reaches the command; then these set, replacing any value:
                CI=1 TERM=dumb NO_COLOR=1 FORCE_COLOR=0 NPM_CONFIG_PROGRESS=false PNPM_PROGRESS=false
                PYTHONUNBUFFERED=1 PIP_PROGRESS_BAR=off DOTNET_NOLOGO=1 DOTNET_CLI_TELEMETRY_OPTOUT=1
              for a nested project, before those: every WINTER_* name removed except WINTER_LOG_LEVEL and WINTER_OTEL_*;
              VIRTUAL_ENV removed, and each PATH entry equal to $VIRTUAL_ENV/bin removed; Call gains
              `remove: &[&OsStr]`, names left out of the child environment, applied before `env`
              # the shell expands whatever these hold; nothing claims otherwise
timeout       none
per command, in declared order:
  CmdStarted; CmdOutputLine per line; CmdCompleted with returncode: the exit code, or minus the signal number
  a non-zero returncode -> CommandFailed; the repository's later commands and steps are not attempted
  the shell cannot be started -> CommandNotStarted
# a user command's effects are its author's to make repeatable (product:bootstrap-recovery); a repeated run starts it again
```

```rust
// crates/core/src/programs.rs — the program port gains streaming, for Program::Shell
pub(crate) enum Program { Git, Shell }                      // Shell: /bin/sh
pub(crate) enum Stdin { Null, Inherit }
// Call gains stdin and remove; every existing caller passes Stdin::Null and no names
fn stream(&self, call: &Call<'_>, line: &mut dyn FnMut(&[u8])) -> io::Result<Exit>;   // stderr joined to stdout
```

### Fan-out

```text
# engineering:bootstrap-effect-accounting; reference: modules/workspace/internal/git_ops_service.py:24,164-167,318-336
workers       at most 4 repositories at once, on std::thread::scope; one repository's steps run in order on one worker;
              the ports are shared across them as managed-content.md, Ports across threads, gives
events        each worker sends its InitEvents over a channel; the calling thread hands them to the sink in arrival
              order, so the events of different repositories interleave and only one repository's own events keep their
              order; a sink Err stops the run: no new repository starts, running ones finish, and the run ends Stopped.
              A sink Err is a stdout that cannot be written, so nothing more can be reported there; the exit code is
              the Termination it carries
outcome       a repository's failure is reported and never stops another repository
shared state  the lock upsert of materialize and every file update go through the file port's lock, so concurrent
              repositories, and concurrent invocations, never lose another's write
interrupt     SIGINT ends the invocation as the runtime's handler does for a command that does not defer (runtime.md,
              Interrupts and signals): children in the process group get the signal too, and a clone git abandons on
              the signal is removed by git; a clone killed by a signal git cannot handle is not told apart from a
              complete one, as in the reference, and the next run reports it exists and runs its later steps
```

### Events and text

| Event             | `--json` line (shape C)                                                         | Text                                                 |
| ----------------- | ------------------------------------------------------------------------------- | ---------------------------------------------------- |
| `TargetStarted`   | `{"type":"target_started","target"}`                                            | `→ <target>`                                         |
| `TargetCompleted` | `{"type":"target_completed","target","success"}`                                | `✓ <target> done`; false: stderr `✗ <target> failed` |
| `RepoAction`      | `{"type":"repo_action","repo","location","action","detail"}`                    | by action, below                                     |
| `RepoError`       | `{"type":"repo_error","repo","error"}`                                          | stderr `[<repo>] ERROR: <error>`                     |
| `CmdStarted`      | `{"type":"cmd_started","repo","command"}`                                       | `[<repo>] $ <command>`                               |
| `CmdOutputLine`   | `{"type":"cmd_output_line","repo","line"}`                                      | `[<repo>] <line>`                                    |
| `CmdCompleted`    | `{"type":"cmd_completed","repo","command","returncode"}`                        | non-zero only, stderr `[<repo>] ✗ <command> (exit <returncode>)` |

```text
# text of a RepoAction, by action; reference: modules/workspace/init_reporter.py:57-100
cloned                      [<repo>] cloned to <location>
exists                      [<repo>] exists at <location>
excludes_updated            [<repo>] excludes updated: <detail>
pinned                      [<repo>] pinned at <detail>
workspace_excludes_updated  [<repo>] workspace excludes updated: <detail>
agents_md_updated           [<repo>] AGENTS.md safety rules updated at <location>
```

Reference: `modules/workspace/init_reporter.py:35-163`. A text row on stderr is written through
`diagnostics::notice`, the exception standalone-pins.md, Running `ws update`, makes for a command's own result rows.
The rendering is the domain's, because `ws init` and `ws prune` both write it:

```rust
// crates/core/src/bootstrap.rs — context/architecture/changes.md: logic two commands share lives in a domain module
pub enum TextStream { Stdout, Stderr }
pub fn text(event: &InitEvent) -> Option<(TextStream, String)>;   // None: a CmdCompleted with returncode 0
```

## Flows

### First run, then a repeat

```text
ws init                         ws init (again, nothing changed)
→ projects/                     → projects/
[winter] workspace excludes…    [api] exists at <root>/projects/api      # blocks unchanged: no event
[api] cloned to <root>/…        [api] $ make setup                       # every cmd runs again
[api] $ make setup              [api] …
[api] …                         ✓ projects/ done
✓ projects/ done                → standalone/
→ standalone/                   [lib] exists at <root>/lib
[lib] cloned to <root>/lib      [lib] pinned at tag 1a2b3c4d             # held: the lock is not rewritten
[lib] pinned at tag 1a2b3c4d    ✓ standalone/ done
[winter-extensions] workspace excludes updated: lib
✓ standalone/ done
```

A repeat never clones, never resets a branch except through a pin, and reruns each `cmd`
(`product:reconcile-repositories`).
