---
name: env-command
depends-on:
  - compute-print
code:
  - winter-workspace/crates/core/src/trace.rs::CommandTracer
  - winter-workspace/crates/cli/src/trace.rs::Tracer
  - winter-workspace/crates/core/topics/root.md
---

# The `winter env` command

Adds `winter env SCOPE [--resolve]` of [env-command.md](../spec/env-command.md): the domain entry for the load, the
registry read, and `compute` with `Consumer::Print`; the lines and their quoting written in one write; nothing on stdout
and one diagnostic on stderr on failure; exit 130 for an interrupted lookup; and the `winter.env` attribute of the
command span. Its row is that of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands, and
its stdout the `env` row of
[output-formats.md](../../../../delivered/parallel-workspaces/component-runtime/spec/output-formats.md), which
env-command.md now owns. Delivers `product:scope-env-command`, the inspection bullets of
`product:offline-env-inspection`, the local-configuration bullet of `product:layered-runtime-values`, the print-and-disk
bullet of `product:private-runtime-values`, and the identity bullets of `product:runtime-scope-values` for `winter env`.

## Change

- `scope_env.rs` gains the domain entry for steps 1 to 3; `crates/core/src/commands/env.rs` registers `env SCOPE` with
  `--resolve`, maps a `ConfigError` to `Termination::Config`, an `EnvError` to `Termination::Env`, and
  `LookupFailure::Interrupted` to `Termination::Interrupted`, and writes the lines through `output::text`.
- `quote` follows Python's `shlex.quote` as the spec's Lines give it; nothing else escapes a value.
- `CommandTracer` gains the call that sets one attribute on the command span; the binary's `Tracer` sets it and
  `NoTrace` ignores it; `env` sets `winter.env` by the spec's rule.
- `crates/core/topics/env.md`, referenced from `root`: the lines, `eval "$(winter env SCOPE)"`, masking and `--resolve`,
  that `--resolve` prints real values, and that a `shell = true` entry's own `$NAME` expansion is not prevented
  (`product:private-runtime-values`).

## Acceptance

- Component tests through `InProcess`: `env alpha` over a recorded `alpha` writes the managed lines and the band keys as
  the spec's example shows, exits 0, and writes nothing on stderr; `env workspace` writes no `WINTER_PORT_BASE`.
- Component tests: `env ghost` with `<root>/ghost` a directory exits 1 with "no env by that name is registered" on
  stderr and no byte on stdout; a recorded index at the buffer slot exits 1 the same way, naming the fault.
- A component test: a key in `.winter/config.local.toml` replaces the shared file's value of that key.
- Component tests: without `--resolve` a command entry prints `'<unresolved:command>'` and starts no program; with it
  the value prints; a failing lookup exits 1 with its first line and details on stderr and nothing on stdout.
- Unit tests of `quote`: `''` for empty, `a/b:c` unchanged, `it's` as `'it'"'"'s'`, and a value holding a newline kept
  inside one quoted string.
- An executable test in `crates/cli/tests`: under `sh -eu`, `eval "$(winter env alpha --resolve)"` sets each variable to
  its value byte for byte, values holding `'`, `$`, a space, and a newline included; for an unknown scope the shell sees
  an empty string and status 1.
- An executable test: a `--resolve` lookup running `sleep 60` given SIGINT exits 130 with nothing on stdout, and its
  process is gone.
- A component test: the run writes no file under the workspace, as a recording file port shows.
- A tracer test: `env alpha/x` sets `winter.env` to `alpha`; `env workspace` and `env /x` set none.
- `cargo dev gate slice` exits 0.
