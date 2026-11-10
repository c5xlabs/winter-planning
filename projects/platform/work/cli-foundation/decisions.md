# Winter CLI foundation decisions

## Release build baseline (decision:cli-foundation/release-build-baseline)

**Decider:** engineering

**Open.** The minimum supported operating-system versions, Linux libc baseline, and CI runners that execute package
smoke tests for the agreed targets; this blocks finalizing `engineering:release-artifacts` and
`engineering:homebrew-distribution`.

## Development sample (decision:cli-foundation/development-sample)

The foundation includes a dev-only Rust sample executable launched through the platform CLI, so the first module
boundary is demonstrated by a real process handoff rather than a mock internal call. Its development command is
`winter sample …`; the sample executable and its command registration are excluded from released installations.

## Release targets (decision:cli-foundation/release-targets)

Initial prebuilt releases and Homebrew installation support macOS and Linux, each on x86_64 (AMD64) and ARM64 (aarch64),
covering the four developer-machine and server targets without adding Windows or 32-bit support. Every target must run
the packaged executable's help and version smoke tests; cross-compilation alone is not sufficient verification.

## Bare invocation (decision:cli-foundation/bare-invocation)

Running `winter` without arguments displays the same help as `winter --help` and exits successfully, so discovering the
interface requires no prior knowledge of its options.

## Standalone command (decision:cli-foundation/standalone-command)

The foundation is the platform's own executable, not a wrapper or fallback into the Python Workspace CLI, so its first
interaction can be established without coupling the two implementations. It follows `decision:launcher-language` and
limits dispatch to the development sample rather than introducing installed-product discovery or version selection.
