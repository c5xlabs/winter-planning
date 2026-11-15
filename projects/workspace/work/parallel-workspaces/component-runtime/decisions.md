# Component runtime decisions

## Fixture helper guarantees (decision:parallel-workspaces/component-runtime/fixture-helper-guarantees)

**Decider:** program lead, under the owner's constraint that tests never touch anything outside their tree

The git fixture helpers themselves refuse, before any effect and with symbolic links resolved, every path that does not
lie in the tree, refuse the tree's root as a repository, and name SHA-1 objects and the files ref backend on every
repository they create. A resolved check held only by the scenario fixture builder would leave component tests a weaker
boundary, and formats left to git's defaults would make the installed git's version part of every recorded object id.

## Git discovery stops at the isolated tree (decision:parallel-workspaces/component-runtime/git-discovery-containment)

**Decider:** program lead, under the owner's constraint that tests never touch a real workspace or repository

A tree is refused when its parent lies inside a git repository, and the pinned environment also sets
`GIT_CEILING_DIRECTORIES` so git's upward search for a repository stops at the tree. Either alone leaves a path open:
the refusal sees only the directories that exist when the tree is created, and the ceiling binds only git calls made
under the pinned environment.

## Gate scope (decision:parallel-workspaces/component-runtime/gate-scope)

**Decider:** product owner

The gates are Cargo only: the format check, Clippy with warnings denied, and the tests, with the recorded scenarios
added in the full gate. No Markdown or workflow lint step joins either gate, because those tools are installed outside
Cargo and the gates run on nothing but the pinned toolchain.

## Test harness provenance (decision:parallel-workspaces/component-runtime/harness-provenance)

**Decider:** product owner

The test harness is written fresh in this repository from the platform harness's pattern — an isolated tree, a pinned
environment, a bounded process runner — and no platform source is copied or adapted. The platform repository carries no
licence and this repository is FSL-1.1-ALv2, so the pattern is taken and the text is not.

## Command registration by file location (decision:parallel-workspaces/component-runtime/command-registration)

**Decider:** engineering

A command is one file under the library's command directory, built with the argument parser's builder interface; a build
script derives each command path from its file path and generates the module declarations and the registry, so adding a
command edits no tracked file. A hand-kept or committed registry makes every parallel command change collide in one
file, link-time registration hides a missing command until run time, and the parser's derive interface needs one shared
enum.

## Four packages, one library (decision:parallel-workspaces/component-runtime/crate-layout)

**Decider:** engineering

The repository is one Cargo workspace of four packages: the binary, one library holding every command and domain module,
test support, and the development driver. A package per domain was rejected for now because every new package edits the
shared workspace and binary manifests, and module boundaries inside one library are enough until a domain needs its own
dependency set.

## Gates run plain Cargo through one driver (decision:parallel-workspaces/component-runtime/gate-driver)

**Decider:** engineering

Each gate is one subcommand of a Rust development binary reached through a Cargo alias, because an alias cannot chain
commands and a shell script would be a second language to lint. Recorded scenarios are one test target of the binary
package selected by a Cargo feature that changes no compiled code, which keeps them out of the slice gate and gives them
the built binary's path from Cargo.

## One CI workflow that runs the full gate (decision:parallel-workspaces/component-runtime/ci-workflow)

**Decider:** engineering

The reference tool's CI workflow is deleted with the reference tool, and the repository carries one workflow whose only
build step is the full gate command, so CI and a local run cannot drift. Keeping no workflow was rejected because
`engineering:reproducible-build` requires CI to run the same checks; the workflow is not exercised until the branch is
pushed.

## Root documents kept through the wipe (decision:parallel-workspaces/component-runtime/retained-root-documents)

**Decider:** engineering

`LICENSE` and `PRINCIPLES.md` stay on the Rust branch unchanged: the licence covers the repository whatever its
language, and the principles state why the product exists without naming the reference tool, its commands, or its files.
Every other root document is rewritten because each one instructs the reader to install or run the reference tool.

## Git failure rendering (decision:parallel-workspaces/component-runtime/git-failure-rendering)

**Decider:** product owner

A failed git call's text reaches stderr once, in the repository error that reports it; it appears additionally as a log
record only when logging is turned up with `-v` or `WINTER_LOG_LEVEL`. The reference tool always writes the failure a
second time as a bare log line, so printing it once is a registered deviation.

## Internal failure rendering (decision:parallel-workspaces/component-runtime/internal-failure-rendering)

**Decider:** product owner

An unexpected failure, a panic included, ends with a diagnostic on stderr and exit 1, where the reference tool prints a
traceback and exits 1. A backtrace follows only when `RUST_BACKTRACE` is set, which is a local development aid.

## Program name (decision:parallel-workspaces/component-runtime/program-name)

**Decider:** product owner

The binary is `winter-workspace` and names itself so in usage, help, and `--version`, whatever path invoked it. The
`config.lock` header comment and trace span names keep the reference tool's literal `winter`, because committed files
and trace consumers carry them.

## Launcher-only surface is absent (decision:parallel-workspaces/component-runtime/launcher-only-surface)

**Decider:** product owner

Commands, options, and files that exist only in the reference tool's bash launcher, such as `init` and `--winter=PATH`,
are not part of the binary; the command surface spec lists each one as deliberately absent. `WINTER_INVOCATION_CWD` and
`WINTER_LAUNCH_TIME`, which the launcher sets and the reference tool itself reads, stay inputs the binary honours with
the reference fallback when unset, though nothing in this program sets them.

## Help rendering (decision:parallel-workspaces/component-runtime/help-rendering)

**Decider:** engineering

Help goes to stdout with exit 0 for `--help`, and to stderr with exit 2 for a bare group. It lists the commands and
options the command surface spec names and omits hidden ones; its wording, order, and layout are the argument parser's
own.

## Error rendering (decision:parallel-workspaces/component-runtime/error-rendering)

**Decider:** engineering

Errors are plain text on stderr in every mode, each naming the subject its error kind carries. `--json` adds no error
envelope; consumers decide success from the exit code.

## Exit-code overloads (decision:parallel-workspaces/component-runtime/exit-code-overloads)

**Decider:** engineering

`ws status` exits 1 for a dirty or drifted worktree and 2 when the command itself fails, and service commands return the
provider's code unmodified. An interrupt exits 130 only on the service paths and 1 everywhere else.

## JSON encoding (decision:parallel-workspaces/component-runtime/json-encoding)

**Decider:** engineering

JSON output is any valid JSON encoding of the reference tool's document: the same keys, values, nesting, and event
kinds, with one document per line where the shape is a stream. Whitespace, key order within an object, character
escaping, and the wording of a human message carried as a string value are not contract; such a value keeps its place
and type.

## JSON-mode purity (decision:parallel-workspaces/component-runtime/json-mode-purity)

**Decider:** engineering

Where the reference tool writes a plain-text line to stdout under `--json`, the binary writes a plain-text line there
too, in its own wording. The output formats spec lists every such command.

## Colour policy (decision:parallel-workspaces/component-runtime/color-policy)

**Decider:** engineering

Colour follows the reference tool's terminal detection, and `NO_COLOR` is ignored.
