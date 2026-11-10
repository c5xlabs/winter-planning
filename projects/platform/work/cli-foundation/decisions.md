# Winter CLI foundation decisions

## Canonical context setup (decision:cli-foundation/canonical-context-setup)

**Decider:** product owner and engineering

Launcher-specific architecture, build commands, and verification guidance live in `./context` inside `c5xlabs/winter`,
so those facts change alongside the implementation they describe. The agent entry point routes contributors and
reviewers to that context and the shared conventions in `c5xlabs/winter-context` under `winter-canon`, without copying
shared rules into application-owned files.

## Sample process contract (decision:cli-foundation/sample-process-contract)

**Decider:** engineering

The development CLI replaces itself with the separately built sample using Unix `exec`, preserving the process ID and
avoiding a waiting parent or signal-forwarding machinery on macOS and Linux. It forwards sample arguments unchanged,
including option-like and non-UTF-8 values, preserves the working directory, streams, and environment except for
removing the development-only sample-path variable, and leaves exit status and signal termination to the sample;
replacement failure reports a diagnostic on stderr and exits unsuccessfully.

## Sample build isolation (decision:cli-foundation/sample-build-isolation)

**Decider:** engineering

A non-default `dev-sample` Cargo feature compiles sample-command registration and launch wiring into the development
CLI, while the sample is built as a separate executable whose absolute path the development harness supplies through a
development-only environment variable, without PATH discovery or a dependency on its implementation. Local tests cover
feature-enabled and normal production builds and prove that environment variables and PATH entries cannot enable sample
dispatch when the feature is disabled; artifact packaging belongs to `epic:release-automation`.

## Architecture enforcement (decision:cli-foundation/architecture-enforcement)

**Decider:** engineering

The foundation uses a Cargo workspace with separate platform-library, thin CLI, test-harness, and independent sample
crates: the CLI and harness depend on the platform library, production never depends on test/sample crates, and service
tests launch the built executables. Rust visibility and Cargo cycle detection, plus a small Cargo-metadata
dependency-allowlist check, enforce these boundaries without custom source analysis or speculative extension frameworks;
exact crate names, libraries, and internal modules are specified in the spec.

## Development sample (decision:cli-foundation/development-sample)

The foundation includes a dev-only Rust sample executable launched through the platform CLI, so the first module
boundary is demonstrated by a real process handoff rather than a mock internal call. Its development command is
`winter sample …`; normal production builds omit the command registration and the sample is not a distributed product.

## Bare invocation (decision:cli-foundation/bare-invocation)

Running `winter` without arguments displays the same help as `winter --help` and exits successfully, so discovering the
interface requires no prior knowledge of its options.

## Standalone command (decision:cli-foundation/standalone-command)

The foundation is the platform's own executable, not a wrapper or fallback into the Python Workspace CLI, so its first
interaction can be established without coupling the two implementations. It follows `decision:launcher-language` and
limits dispatch to the development sample rather than introducing installed-product discovery or version selection.
