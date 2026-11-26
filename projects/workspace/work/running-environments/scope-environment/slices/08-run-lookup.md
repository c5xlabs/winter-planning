---
name: run-lookup
depends-on:
  - scope-identity
  - resolve-literals
  - lookup-output-formats
  - process-group-calls
  - epic:parallel-workspaces/workspace-bootstrap
---

# Running one command entry

Adds `run_lookup` of [value-lookups.md](../spec/value-lookups.md), Domain: the command line, the child, and the failures
of one command entry under `Lookups::Run`, its output read by `lookup-output-formats`. Delivers the execution bullets of
`product:external-value-resolution`, the lookup half of the caller-values bullet of
`product:complete-launch-environment`, the diagnostic and tracing bullets of `product:private-runtime-values`,
`engineering:bounded-value-commands`, and `engineering:process-execution` for this seam, under
`decision:running-environments/scope-environment/lookup-execution-policy`,
`decision:running-environments/scope-environment/inherited-environment`, and
`decision:running-environments/scope-environment/resolution-compatibility`, whose word split before substitution is a
registered deviation.

## Change

- `shell = false`: the declared command is split as Python's `shlex.split` in POSIX mode without comments, failing as
  `Tokenize` or `Empty`; references are then rendered inside each word with `resolve-literals`' render, and the words
  run as `Program::Argv`.
- `shell = true`: the declared command is rendered whole and run as `Program::Shell`.
- The call: `cwd` the workspace root, `Stdin::Null`, `remove` the five managed names of `scope-identity`, `env` the
  request's `visible` variables, a fixed 30 s timeout, `Group::Own`, and the invocation's `Interrupts`.
- Its ending: a failed start is `Start` with the operating system's message alone; a non-zero exit or a signal is `Exit`
  with stderr trimmed; the timeout is `Timeout`; `pending()` after the call is `Interrupted`; exit 0 is read by the
  format and filtered. A succeeding lookup's stderr is discarded.
- A lookup opens no span and writes no log record; `LookupError` carries the declared command, never the rendered one.
- No caller yet but tests.

## Acceptance

- Component tests with real programs: a raw `printf` entry answers its output; `printf '%s' "${A}"` with `A` visible as
  `x y;rm` passes one argument, `x y;rm`, and runs nothing else; a shell entry `echo "$HOME"` sees the snapshot's
  `HOME`.
- A component test: with the snapshot holding `WINTER_ENV=beta` and `WINTER_PORT_BASE=9`, and `visible` holding the
  workspace scope's variables, the child sees no `WINTER_PORT_BASE` and `WINTER_ENV=workspace`.
- Component tests: the child's working directory is the root, and its stdin reads empty.
- Component tests of failures: `'unbalanced` is `Tokenize`; `""` is `Empty`; an absent program is `Start` whose cause
  holds no program path; `sh -c 'echo no >&2; exit 3'` is `Exit` with code 3 and stderr `no`, and its rendering shows
  the cwd, exit 3, and `no`; a JSON entry printing `[]` is `Output(JsonNotObject)` with exit 0.
- A unit test over a recording program port: every call carries a 30 s timeout and `Group::Own`, whatever the entry
  declares; a call the port ends as its timeout is `Timeout` with exit -9 (`process-group-calls` proves the group kill).
- A component test: an interrupt noted during a sleeping lookup answers `Interrupted`.
- A unit test over a recording tracer: a lookup opens no operation span, and no log record holds the command, a visible
  value, or the output.
- `cargo dev gate slice` exits 0.
