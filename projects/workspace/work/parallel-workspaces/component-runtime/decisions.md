# Component runtime decisions

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
