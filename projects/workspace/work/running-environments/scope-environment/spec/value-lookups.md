# Value lookups

Running one command entry under `Lookups::Run`: the command line, the child process, its output formats, and its
failures. When an entry runs is [resolution.md](./resolution.md), The fixpoint; which operations run lookups at all is
[consumers.md](./consumers.md). The program port is the one of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md), as
`epic:parallel-workspaces/workspace-bootstrap` extends it with `Program::Shell`, `Stdin`, and `remove`. Reference
locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the
binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/scope_env/lookup.rs — engineering:bounded-value-commands, engineering:value-provenance
pub struct LookupRequest<'a> {
    pub band: &'a str,                      // "env.workspace.vars", "env.feature.vars", "env.<name>.vars"
    pub key: &'a str,
    pub entry: &'a EnvCommand,              // command as declared, format, shell, exports
    pub visible: &'a [(String, String)],    // the entry's visible variables (resolution.md, Visibility)
}
pub fn run_lookup(
    programs: &dyn RunPrograms,
    interrupts: &Interrupts,
    root: &Path,
    request: &LookupRequest<'_>,
) -> Result<Vec<(String, String)>, LookupError>;   // filtered output, in output order
/// A finished lookup's stdout read in the entry's format, then filtered (Formats); pure, starts nothing.
pub fn read_output(entry: &EnvCommand, key: &str, stdout: &[u8]) -> Result<Vec<(String, String)>, LookupFailure>;

#[derive(Debug, thiserror::Error)]
pub struct LookupError {
    pub band: String,
    pub key: String,
    pub command: String,                    // the declared command, never the rendered one
    pub cwd: PathBuf,                       // the workspace root
    pub failure: LookupFailure,
}
pub enum LookupFailure {
    Tokenize,                               // unbalanced quote or trailing backslash in the declared command
    Empty,                                  // no words
    Start { cause: String },                // the operating system's message alone, without the program path
    Exit { exit: Exit, stderr: String },    // non-zero code or a signal; stderr trimmed
    Timeout,                                // exit -9
    NotUtf8,                                // stdout is not UTF-8
    Output(OutputFault),                    // exit 0, output not of the declared format
    Interrupted,                            // an interrupt was pending when the call returned; compute answers it
                                            // as EnvError::Interrupted (consumers.md)
}
pub enum OutputFault {
    DotenvLine { line: usize },             // 1-based; the line's text is never carried
    JsonSyntax { cause: String },           // the parser's message: position only
    JsonNotObject { found: &'static str },  // "array", "string", "number", "boolean", "null"
    JsonKey { key: String },                // not ^[A-Za-z_][A-Za-z0-9_]*$
    JsonNotScalar { key: String },          // an object or array value
}

// crates/core/src/programs.rs — the program port gains
pub enum Program { Git, Shell, Argv }       // Argv: args[0] is the program; the rest are its arguments
pub enum Group { Inherit, Own, Shared }     // Call.group; every existing call is Inherit
pub struct Call<'a> { /* … as child-processes.md and reconciliation.md give it */ pub group: Group,
                      pub interrupts: Option<&'a Interrupts> }   // Some only with Group::Own or Group::Shared
pub trait RunPrograms { /* … */ fn has_controlling_terminal(&self) -> bool { false } }   // The child, group
impl OsPrograms { pub fn with_controlling_terminal(self, has: bool) -> Self; }          // answers `has`

// crates/core/src/process.rs — the invocation gains
pub struct Invocation<'a> { /* … as runtime.md gives it */ pub has_controlling_terminal: bool }
```

Visibility: `run_lookup`, `read_output`, and `LookupRequest` are `pub` inside their private module and re-exported by
name from `lib.rs`, the names of `scope_env/lookup.rs` on a `pub use` line of their own, with every type their
signatures name: `LookupError`, `LookupFailure`, and `OutputFault`; the program port's `Exit`, `Program`, and `Call` are
already `pub`, as standalone-pins.md, Domain, makes them, and `Group` joins them on the `programs` line as
`ProcessGroup`, since `fan_out::Group` of `epic:parallel-workspaces/coordinated-git` holds `Group` at the crate root
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands each tests it
before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`). Everything else here stays `pub(crate)`. standalone-pins.md is
[../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md](../../../../delivered/parallel-workspaces/reproducible-inputs/spec/standalone-pins.md).

## Interfaces

### The command line

```text
# reference: modules/workspace/internal/subprocess_command_entry_runner.py:106-190,
#            modules/workspace/env_band_resolver_service.py:1035-1099
shell = false   the declared command is split into words first, by POSIX shell quoting as Python's shlex.split
                (posix, no comments): blanks separate; '…' is literal; "…" keeps \\ \" \$ \` and \newline escapes;
                \ outside quotes escapes the next character. A split failure -> Tokenize; no words -> Empty.
                References are then rendered inside each word (resolution.md, References), so a substituted value
                fills exactly the word it was written in and never adds, removes, or re-quotes a word
                registered deviation, decision:running-environments/scope-environment/resolution-compatibility:
                the reference renders the whole string and then splits it
                program  Program::Argv, args[0] looked up on the child's PATH as find_program does; a word holding
                         "/" is that path, relative to the root
shell = true    the declared command is rendered, then run as Program::Shell: /bin/sh -c <rendered>
                a reference to a command-held key was already refused (ResolutionError::ShellReference); the shell's
                own $NAME expansion of the child environment is not prevented (product:private-runtime-values)
```

### The child

```text
# engineering:bounded-value-commands
cwd       the workspace root
stdin     Stdin::Null
env       the environment OsPrograms gives every child (child-processes.md), with
          remove  WINTER_ENV, WINTER_ENV_INDEX, WINTER_PORT_BASE, WINTER_WORKSPACE_PORT_BASE, WINTER_SERVICE_PREFIX
          env     the entry's visible variables, laid over
          registered deviation, decision:running-environments/scope-environment/inherited-environment: the
          reference lays the variables over its whole process environment, so a sourced scope's stale managed
          values reached the lookup
timeout   30 s, fixed; no option, variable, or entry field changes it
group     chosen by programs.has_controlling_terminal(), the fact the binary detects once, as it does
          stdout_is_terminal, by whether open("/dev/tty", O_RDWR | O_NOCTTY) succeeds, carries as
          Invocation.has_controlling_terminal, and hands the host's OsPrograms through with_controlling_terminal;
          a test injects it through the same port, so neither case needs a real terminal
          no controlling terminal, as under an agent or CI
                   Group::Own: the child leads a new process group (setpgid(0, 0) before exec). On the deadline
                   the adapter sends SIGKILL to the whole group and reaps the child -> Timeout
                   registered deviation, decision:running-environments/scope-environment/lookup-execution-policy:
                   the reference kills the direct child only, and a shell entry's grandchildren outlive it
          a controlling terminal
                   Group::Shared: the child stays in Workspace's process group, as the reference's does, so it can
                   read the terminal and prompt, as a passphrase or MFA prompt does. On the deadline the adapter
                   sends SIGKILL to the direct child alone and reaps it -> Timeout; its descendants may survive, as
                   the reference's do
interrupt the call defers the interrupt for its duration; the adapter waits in slices of at most 100 ms and, once
          pending() is true, sends SIGKILL to the group (Group::Own) or to the direct child alone (Group::Shared),
          reaps the child, releases, and answers Interrupted
          a child in its own group does not receive the SIGINT a terminal or a caller sends Workspace's group, and
          one in Workspace's group may ignore it, so without the deferral the handler's _exit would leave the
          lookup running; the deferral is the means of the kill of
          decision:running-environments/scope-environment/lookup-execution-policy, and the invocation still ends as
          the reference's does: aborted, the aborted notice, exit 1 (consumers.md, Failure by consumer). The frozen
          runtime.md, which lists the service paths as the only deferrals, is not edited: this leaf adds the lookup
          call beside them, and changes nothing a user sees of an interrupt
          runtime.md = ../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md
output    stdout and stderr captured whole; stderr of a lookup that succeeds is discarded
```

### Formats

```text
# reference: modules/workspace/env_band_resolver_service.py:463-572
stdout must be UTF-8, else NotUtf8
raw      [(key, stdout trimmed of white space)]
dotenv   lines split at "\n", a trailing "\r" removed; each line trimmed
           empty, or starting "#"                 skipped
           otherwise split at the first "=", else DotenvLine { line }; the name, trimmed, matches
           ^[A-Za-z_][A-Za-z0-9_]*$, else DotenvLine { line }. "FOO" and "export FOO=1" are such faults
           value: trimmed, then one surrounding pair of the same quote, ' or ", removed; nothing else unquoted,
           no escapes, no inline comment removed
           a repeated name: the last line wins, at the place of its first
json     one JSON value (RFC 8259; NaN and Infinity are syntax faults) that is an object, else JsonNotObject
           a key not matching the name pattern -> JsonKey; an object or array value -> JsonNotScalar
           string -> its content; true, false, null -> "true", "false", "null"; a number -> its text as the output
           wrote it
           decision:running-environments/scope-environment/resolution-compatibility: the reference's Python
           spellings ("True", "None", "1e+16") are not kept
           a repeated key: the last wins, at the place of its first
filter   in order, drop a name starting "_", a name starting "WINTER_", and a name outside `exports` when the entry
         declares it; raw entries included (product:external-value-resolution)
```

### Failure rendering

```text
first line   "<band> key '<key>': command `<command>` <what>", what by failure:
             Tokenize "could not be tokenized"; Empty "rendered to an empty command line"; Start "could not be run";
             Exit "failed"; Timeout "timed out after 30s"; NotUtf8 "output could not be decoded as text";
             DotenvLine "output line <n> is not a valid dotenv KEY=VALUE line"; JsonSyntax "output is not valid
             JSON — <cause>"; JsonNotObject "output must be a flat JSON object, got <found>"; JsonKey "output JSON key
             '<key>' is not a valid env-var identifier"; JsonNotScalar "output JSON key '<key>' is not a flat scalar"
             Interrupted has none: it is never rendered, since compute answers it as EnvError::Interrupted
details      as a RepoError's (results-and-exit-codes.md): cwd; exit (Exit, Timeout -9, Output 0); stderr (Exit's,
             or Start's cause). A failing lookup's stderr is shown as it wrote it
             (decision:running-environments/scope-environment/persistent-value-exposure)
never        a rendered command, a substituted value, or an output line
tracing      a lookup opens no span and writes no log record holding its command line, its environment, or its
             output; the command span carries none of them (engineering:observability-isolation)
```
