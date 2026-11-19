# Workspace decisions

## Provider platform support (decision:supported-systems)

**Decider:** product owner and engineering

**Open.** Choose supported service-provider combinations and external runtime prerequisites within the shared CLI
baseline governed by `engineering:cli-runtime-baseline`; this blocks provider coverage acceptance for
`epic:rust-adoption/workspace-release`, not portable local development. Minimum OS/libc versions are settled once by
`decision:platform/release-automation/release-build-baseline`, not separately for Workspace.

## TOML parity is parity of meaning (decision:toml-parity)

**Decider:** product owner

Parity for every TOML file the binary reads or writes is at the deserialized model: file content both libraries accept
means the same configuration, lock, or state in both tools. How a file is parsed and serialised is the Rust TOML
library's — which text it accepts or refuses, its error texts, escapes, and the key, table, and blank-line layout an
edit leaves — so a difference of that kind is no change of function and never a registered deviation, and an edit keeps
what the user wrote only as far as the library does. A made decision may fix lines beyond the library, as
`decision:parallel-workspaces/repository-catalog/removal-comment-extent` does; it is a registered deviation wherever the
reference's lines differ, and no other layout difference is one. This overtakes the clause of
`decision:parallel-workspaces/workspace-configuration/write-conflicts` that edits preserve comments, order, and
unrelated content as in the reference tool. The owner cares about the format of the data, not about the faults of
Python's TOML libraries.

## Development component selection (decision:development-component-selection)

**Decider:** product owner and Platform and Workspace engineering

Engineers and agents build Rust Workspace in their feature-environment worktree and invoke its `winter-workspace`
executable directly, leaving the installed launcher, release selections, and version cache unchanged. The
`winter-workspace` code repository owns discoverable agent/runtime guidance for this workflow, distinguishing direct
Rust invocation from the Python-only `--winter=PATH` compatibility path retained under
`decision:platform/singleton-launcher-succession`, not adding a Rust environment-selection or build-and-run mechanism.

## Functional parity, not byte parity (decision:functional-parity)

**Decider:** product owner

The binary matches the reference tool in function: commands, arguments, and options; configuration files, keys, and
their meaning; on-disk and git state after a command; exit-code meanings; the information in `--json` output; and the
contracts toward other programs. Wording and layout of human output, help, error and warning texts, causes embedded from
a library or the operating system, JSON whitespace and escaping, and behaviour that exists only as an artifact of Python
or its libraries follow idiomatic Rust and need no individual decision.

## Visible gaps are asked, not fixed (decision:visible-gap-policy)

**Decider:** product owner

The binary reproduces the reference tool's observable function by default, defects included; hardening no caller can
observe, such as atomic writes and locks, is allowed. Each change of function — what a command does, accepts, refuses,
writes, or reports as data — is put to the product owner individually and, when accepted, recorded as a decision of the
epic that owns the behaviour; a difference of text or format alone falls under `decision:functional-parity`.

## Command vocabulary (decision:command-vocabulary)

**Decider:** product owner and engineering

The binary mirrors the reference tool's command tree, option names, and per-command `--json` shapes, with exit codes
`0`, `1`, `2`, `126`, `130`, and provider passthrough. The contract is the component runtime's command surface, output
formats, and results specs.

## Configuration transition (decision:configuration-transition)

**Decider:** product owner and engineering

The binary reads and writes `.winter/config.toml`, `.winter/config.local.toml`, `.winter/config.lock`, and
`.winter/state.toml` in the reference tool's formats with no conversion step. A component pin shared with Platform is
not part of Workspace configuration.

## Observability shutdown budget (decision:observability-shutdown)

**Decider:** engineering

Exporter shutdown waits at most 100 ms, as the reference tool does. The cap does not cover name resolution performed
before the exporter starts.

## Initiatives toward parity (decision:parity-initiative)

The rewrite has four outcome-based investments: `initiative:parallel-workspaces`, `initiative:running-environments`,
`initiative:agent-ready-operation`, and `initiative:rust-adoption`. Rough parity is their shared
`milestone:rust-workspace-parity`, not an initiative containing every capability; epic scope and requirements remain
intact while the folders and references follow these ownership boundaries.

## Parity boundary (decision:rough-parity)

Parity is functional, as `decision:functional-parity` defines it, and is kept under `decision:visible-gap-policy`.
Python's internal structure and its in-process plugin API are not reproduced, and TOML files match in meaning only,
under `decision:toml-parity`; the only other differences of function are the registered deviations, each a made decision
of the epic that owns the behaviour and marked as one in that epic's spec. The component runtime's command surface spec
lists those made before it was delivered.

## Rust replacement without Python delegation (decision:rust-replacement)

- The replacement is a Rust `winter-workspace` component in the `winter-workspace` code repository, independently usable
  and invoked through the platform's `winter workspace` group. It does not forward unfinished commands to the Python CLI
  or require Python for first-party Workspace behavior; Platform's separate, temporary Python launch adapter under
  `decision:platform/singleton-launcher-succession` preserves existing workspaces without making Python part of the Rust
  component, and user-owned external programs remain the user's runtime choice.
- The replacement is one Rust binary, `winter-workspace`, in the `winter-workspace` code repository, invoked directly.
  It reads no `component.toml`, `.winterrc`, or launcher handoff metadata, never forwards a command to the Python CLI,
  and never executes Python for first-party behaviour; user-owned external programs remain the user's runtime choice.

## Specs before slices (decision:requirements-first)

Full specs are written from the reference tool's observable behaviour before any implementation slice is cut. An area
whose behaviour is not yet settled keeps an owned open decision instead of a guessed contract.
