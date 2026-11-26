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
pub(crate) struct LookupRequest<'a> {
    pub band: &'a str,                      // "env.workspace.vars", "env.feature.vars", "env.<name>.vars"
    pub key: &'a str,
    pub entry: &'a EnvCommand,              // command as declared, format, shell, exports
    pub visible: &'a [(String, String)],    // the entry's visible variables (resolution.md, Visibility)
}
pub(crate) fn run_lookup(
    programs: &dyn RunPrograms,
    interrupts: &Interrupts,
    root: &Path,
    request: &LookupRequest<'_>,
) -> Result<Vec<(String, String)>, LookupError>;   // filtered output, in output order

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
    Interrupted,                            // ends the invocation as Termination::Interrupted, exit 130
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
pub enum Group { Inherit, Own }             // Call.group; every existing call is Inherit
pub struct Call<'a> { /* … as child-processes.md and reconciliation.md give it */ pub group: Group,
                      pub interrupts: Option<&'a Interrupts> }   // Some only with Group::Own
```

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
group     Group::Own: the child leads a new process group (setpgid(0, 0) before exec). On the deadline the adapter
          sends SIGKILL to the whole group and reaps the child -> Timeout
          registered deviation, decision:running-environments/scope-environment/lookup-execution-policy: the
          reference kills the direct child only, and a shell entry's grandchildren outlive it
interrupt the call defers the interrupt for its duration (runtime.md, Interrupts and signals, gains this deferral
          beside the service paths); the adapter waits in slices of at most 100 ms and, once pending() is true,
          sends SIGKILL to the group, reaps the child, releases, and answers Interrupted
          (a child in its own group no longer receives the terminal's SIGINT)
output    stdout and stderr captured whole; stderr of a lookup that succeeds is discarded
```

### Formats

```text
# reference: modules/workspace/env_band_resolver_service.py:463-572
stdout must be UTF-8, else NotUtf8
raw      [(key, stdout trimmed of white space)]
dotenv   lines split at "\n", a trailing "\r" removed; each line trimmed
           empty, or starting "#"                 skipped
           otherwise split at the first "="; the name, trimmed, matches ^[A-Za-z_][A-Za-z0-9_]*$, else
           DotenvLine { line }. "export FOO=1" is such a fault
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
details      as a RepoError's (results-and-exit-codes.md): cwd; exit (Exit, Timeout -9, Output 0); stderr (Exit's,
             or Start's cause). A failing lookup's stderr is shown as it wrote it
             (decision:running-environments/scope-environment/persistent-value-exposure)
never        a rendered command, a substituted value, or an output line
tracing      a lookup opens no span and writes no log record holding its command line, its environment, or its
             output; the command span carries none of them (engineering:observability-isolation)
```
