# The `env` command

`winter env SCOPE [--resolve]`: one scope's variables as lines a POSIX shell evaluates. Its arguments are the `env` row
of [command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands;
its stdout is the `env` row of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), Results another
program reads, which this leaf now owns; its failures are the `env` row of
[results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md).
The computation is `compute` with `Consumer::Print` ([consumers.md](./consumers.md)). Reference locations are paths
under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`. Message wording is the binary's own
(`decision:functional-parity`).

## Interfaces

```text
# crates/core/src/commands/env.rs — product:scope-env-command; reference: modules/workspace/env_command.py:47-86
order
  1. root discovery and the load (configuration-model.md, Load); a failure is its ConfigError, exit 1
  2. the registry (state_registry::read); a failure is its ConfigError, exit 1
  3. compute(SCOPE, Consumer::Print { resolve: --resolve }); a failure is Termination::Env, exit 1
       unknown name          EnvError::UnknownScope, also when a directory of that name exists
       recorded index fault  EnvError::RecordedIndex
       resolution, lookup    EnvError::Resolution; a lookup failure carries its details (value-lookups.md)
       interrupted lookup    EnvError::Interrupted is not Termination::Env: it ends as
                             Termination::Aborted { interrupted: true }, the aborted notice, exit 1, nothing on
                             stdout, as results-and-exit-codes.md gives an interrupt outside the service paths and the
                             reference's click Abort does (cli.py:434-436); the lookup's group is killed first
                             (value-lookups.md, The child)
  steps 1 to 3 are one domain entry of scope_env, which the command file calls and whose failure it maps
  (context/architecture/changes.md, A command: a command file names neither config_load nor state_registry); the
  entry fails with EnvCommandError { Config(Box<ConfigError>), Env(Box<EnvError>) }, of scope_env.rs and re-exported
  from lib.rs, which the command file maps: Config -> Termination::Config, Env(Interrupted) ->
  Termination::Aborted { interrupted: true }, any other Env -> Termination::Env
  4. every line is built in memory, then written to stdout in one write and flushed; exit 0
     a failed write ends as Termination::BrokenPipe or the write's error (runtime.md)
stdout on failure   nothing: steps 1-3 write no byte to stdout, so `eval "$(winter env x)"` under sh -eu sees an
                    empty string and the non-zero status
stderr on failure   the one diagnostic of the Termination; no other line
--resolve           Lookups::Run; without it Lookups::Mask, and a command entry's key prints PLACEHOLDER
                    ("<unresolved:command>"), which providers match literally. No notice is written for --resolve
tracing             the command span gets winter.env = the part of SCOPE before the first "/" when that part is not
                    empty and not "workspace", whether or not the name is registered
                    (reference: modules/workspace/env_target.py:76-138); the tracer port (CommandTracer, tracing.md)
                    gains a call that sets one attribute on the open command span, which the binary's tracer applies
                    and NoTrace ignores
```

### Lines

```text
each scope       one line per variable, in ScopeEnv order:            export <KEY>=<quote(value)>\n
                 no other line: the workspace scope prints no WINTER_PORT_BASE and no unset; a stale value in the
                 caller's shell is removed where Winter launches (consumers.md, Launch environment)
KEY              a valid shell name in every case: managed names, band keys (refused at load otherwise,
                 resolution.md), and imported keys (value-lookups.md, Formats)
quote(s)         as Python's shlex.quote:
                   s empty                                   -> ''
                   every byte of s in [A-Za-z0-9_@%+=:,./-]  -> s unchanged
                   otherwise                                 -> ' + s with each ' written as '"'"' + '
                 the value's bytes are kept as they are: no other escaping, newlines included
```

```text
$ winter env alpha
export WINTER_ENV=alpha
export WINTER_ENV_INDEX=1
export WINTER_PORT_BASE=4020
export WINTER_WORKSPACE_PORT_BASE=4000
export WINTER_SERVICE_PREFIX=winter
export API_PORT=4021
export DB_PASSWORD='<unresolved:command>'

$ winter env workspace
export WINTER_ENV=workspace
export WINTER_ENV_INDEX=0
export WINTER_WORKSPACE_PORT_BASE=4000
export WINTER_SERVICE_PREFIX=winter
```
