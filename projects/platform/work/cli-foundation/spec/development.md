# Development commands and test harness

## System

```text
Rustup + host C linker/toolchain + Bash/curl/tar/checksum utility
  -> ./scripts/bootstrap-dev.sh
  -> pinned Rust components and repository-local development tools installed
  -> cargo fetch --locked; dprint plugins cached during bootstrap
  -> offline Cargo development driver
       -> host Cargo/tool subprocesses, preserving build-toolchain environment
       -> built executable paths from Cargo JSON
       -> isolated runtime fixtures, never installed winter
```

| Pin owner             | Initial contents                                                                                        |
| --------------------- | ------------------------------------------------------------------------------------------------------- |
| `rust-toolchain.toml` | Rust `1.99.1`, rustfmt, Clippy; toolchain contract in [system.md](./system.md)                          |
| `Cargo.lock`          | Exact production, test, and sample Rust dependency resolutions                                          |
| `dev-tools.env`       | Exact dprint, rumdl, and actionlint versions; shared by bootstrap and driver preflight                  |
| `dprint.json`         | Markdown `0.29.0`, JSON `0.26.0`, TOML `0.10.0`, pretty_yaml `0.6.0`, shfmt `0.2.0` versioned WASM URLs |
| `.rumdl.toml`         | Checked-in Markdown rules and 120-column prose limit                                                    |

Bootstrap installs the pinned tools and warms Cargo/plugin caches; check tasks never install missing tools or download
dependencies. Missing tools or cache entries fail with the setup prerequisite, not an implicit installation attempt.

```sh
# dev-tools.env: only these literal key=value entries; no expansion or general dotenv syntax.
DPRINT_VERSION=0.64.3
RUMDL_VERSION=0.3.2
ACTIONLINT_VERSION=1.7.12
```

| Bootstrap operation      | Contract                                                                                                                      |
| ------------------------ | ----------------------------------------------------------------------------------------------------------------------------- |
| Entry point              | `./scripts/bootstrap-dev.sh`; invoked explicitly, with network access                                                         |
| Rust installation        | Rustup selects `rust-toolchain.toml`; no second compiler-version setting                                                      |
| Cargo cache              | `cargo fetch --locked` for the checked-in dependency graph                                                                    |
| Rust development tools   | `cargo install --locked --version <pin> --root .tools dprint` and equivalent for rumdl                                        |
| actionlint               | Download the pinned host/architecture release archive, verify its release checksum, extract only the binary into `.tools/bin` |
| Formatter cache          | Warm the versioned dprint plugins during bootstrap                                                                            |
| Repeated invocation      | Reuse matching installed versions; no writes to the user's Cargo bin directory or shell profile                               |
| Routine task tool lookup | Absolute `<workspace>/.tools/bin/<tool>` paths; verify pinned versions and fail on mismatch                                   |

The driver reads the same literal version file as bootstrap and rejects missing, duplicate, or unknown keys. Its tool
checks do not install software or modify PATH globally.

## Domain

```rust
// winter-test-harness: shared fixtures and process ownership, not a production dependency.
pub enum Configuration { Production, Development }
pub enum Tier { Unit, Component, Service }
pub enum Termination { Code(i32), Signal(i32) }
pub enum TestSignal { Interrupt, Terminate }
pub enum Completion {
    Exit,
    SignalAfterStdout { marker: Vec<u8>, signal: TestSignal },
}

pub struct Invocation {
    pub executable: std::path::PathBuf, // absolute, explicitly supplied
    pub args: Vec<std::ffi::OsString>,
    pub cwd: std::path::PathBuf,
    pub env: std::collections::BTreeMap<std::ffi::OsString, std::ffi::OsString>,
    pub stdin: Vec<u8>,
    pub timeout: std::time::Duration,   // finite, positive; default 10 seconds
    pub completion: Completion,
}

pub struct Observation {
    pub spawned_pid: u32,
    pub stdout: Vec<u8>,
    pub stderr: Vec<u8>,
    pub termination: Termination,
}

pub enum FailureKind {
    InvalidInvocation,
    Spawn(std::io::Error),
    Timeout,
    OutputLimit,
    ExitBeforeReadiness,
    Cleanup(std::io::Error),
}

pub struct InvocationFailure {
    pub kind: FailureKind,
    pub executable: std::path::PathBuf,
    pub spawned_pid: Option<u32>,
    pub stdout: Vec<u8>,
    pub stderr: Vec<u8>,
    pub termination: Option<Termination>,
    pub cleanup_error: Option<std::io::Error>,
}

pub fn invoke(invocation: Invocation) -> Result<Observation, InvocationFailure>;
```

| Harness boundary                          | Behavior                                                                                                                                      |
| ----------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------- |
| Runtime child environment                 | `env_clear`; fresh HOME, XDG config/cache/data/state, TMPDIR; explicit fixture PATH and probe variables                                       |
| Build/tool environment                    | Retain Cargo/Rustup locations, linker access, and installed tools; normalize Rust warning flags                                               |
| Runtime child ownership                   | Guard owns the direct child; fixtures inherit the enclosing driver-owned Cargo/test process group and do not detach or daemonize              |
| Stream capture                            | Separate temporary files for stdin/stdout/stderr; no pipe-reader or writer thread joins                                                       |
| Live output observation                   | Read through separately opened descriptors, without sharing the child's file offset                                                           |
| Completion loop                           | Monotonic deadline; poll at intervals no greater than 10 ms; detect output overflow while running                                             |
| Normal completion                         | Reap the direct child before releasing fixture state                                                                                          |
| Timeout, overflow, early return, or panic | Kill/reap the owned direct child; retain bounded output before cleanup                                                                        |
| Readiness action                          | Under the same overall deadline, detect the stdout marker, signal the owned PID once, then await termination; exit before marker is a failure |
| Output limit                              | Poll for more than 1 MiB per stream and terminate on detection; retain at most 1 MiB per stream, not a hard on-disk write quota               |
| Signaling                                 | Address only a positive owned PGID/PID; never signal the driver's own process group                                                           |
| Signal assertions                         | Read `ExitStatusExt::signal()`, not shell-style `128 + signal` status codes                                                                   |

`wait` readiness is detected from its flushed stdout marker before a test sends a signal. Service fixtures include
empty/spaced/option-like/non-UTF-8 arguments, non-UTF-8 executable paths, controlled env/cwd/stdin, missing and
non-executable targets, executable text without a shebang that must fail without running its body, and PATH decoys that
write markers if wrongly executed.

```text
driver process
  -> Cargo/tool child in its own process group G
       -> service test binary, still in G
            -> runtime winter/sample/decoys, still in G; exec preserves membership

driver normal completion, child failure, timeout, SIGINT, or SIGTERM:
  -> cleanup group G, including fixtures whose test process died
  -> reap direct Cargo/tool child; retain diagnostics
```

The driver handles interrupt/termination through a cancellation flag, not default termination before cleanup. Group
cleanup sends `SIGKILL` to the owned positive PGID and performs bounded reap observation; cleanup failures are reported.
Runtime fixture guards never kill their enclosing group. Service tests require the driver-owned scope in addition to
artifact paths, so an outer timeout reaches fixtures without nested process-group registries or a supervisor service.

## Interfaces

```toml
# .cargo/config.toml
[alias]
dev = "run --locked --offline -p winter-test-harness --bin winter-dev --"
```

```text
cargo dev check
cargo dev format [--write]
cargo dev lint
cargo dev architecture
cargo dev build --configuration production|development
cargo dev test unit|component|service --configuration production|development [--filter TEXT]
cargo dev sample -- [ARG...]
```

| Driver command       | Contract                                                                                                                                                             |
| -------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `check`              | One shared local/CI operation list: format check, lint, architecture, both configuration builds and all three test tiers                                             |
| `format`             | rustfmt and dprint check only; `--write` is the sole explicit source-rewriting mode                                                                                  |
| `lint`               | Clippy on all Rust targets in both configurations, rumdl, `actionlint -shellcheck= -pyflakes=`, and `bash -n` on the bootstrap script; every enabled check must pass |
| `architecture`       | Cargo declaration allowlist and its checker self-tests from [system.md](./system.md)                                                                                 |
| `build`              | Build the selected CLI configuration; development also builds the independent sample                                                                                 |
| `test unit`          | Inline library/binary unit targets, not integration targets                                                                                                          |
| `test component`     | Harness `tests/component.rs` against platform public interfaces and controlled process doubles                                                                       |
| `test service`       | Harness `tests/service.rs` against explicitly built CLI/sample artifacts                                                                                             |
| `--filter TEXT`      | Pass a Rust test-name filter to the selected tier; no format/lint sweep or clean rebuild                                                                             |
| `sample -- [ARG...]` | Build development artifacts, then exec the built `winter sample` with the explicit sample path and raw tail                                                          |

| Driver result                                                          | Exit                                                                                                                                         |
| ---------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| All requested checks pass                                              | `0`                                                                                                                                          |
| Check failure, missing prerequisite, timeout, or orchestration failure | `1`                                                                                                                                          |
| Invalid developer-command usage                                        | `2`                                                                                                                                          |
| Sample demonstration                                                   | Driver execs the CLI; subsequent status and signals belong to the replaced executable; failed replacement exits `1` with a driver diagnostic |

The driver is internal development tooling, never installed, published, or registered in `winter` help. It contains the
task implementation inside the existing harness crate rather than adding an xtask crate or a second command runner.

```text
production target directory  = <workspace>/target/verification/production
development target directory = <workspace>/target/verification/development

driver build/check/test/clippy/metadata invocations:
  --locked --offline --no-default-features
  explicit package/target selection; no --workspace --all-features isolation test
  compilation warnings denied; RUSTFLAGS and RUSTDOCFLAGS include -Dwarnings
  inherited CARGO_ENCODED_RUSTFLAGS normalized so it cannot override warning enforcement

rustfmt invocation:
  cargo fmt --all -- --check; omit Cargo dependency/feature flags not accepted by this subcommand

build artifact lookup:
  cargo build --message-format=json ...
  -> compiler-artifact matching package ID + binary target name
  -> exactly one executable path, converted to absolute
  -> never guess target/debug paths or use PATH / cross-package CARGO_BIN_EXE variables

service-test inputs supplied by driver:
  WINTER_TEST_CLI_PATH          = selected built winter executable
  WINTER_TEST_EXPECTED_VERSION  = winter-cli package version from Cargo metadata
  WINTER_TEST_SAMPLE_PATH       = built sample, for development tests only
  WINTER_TEST_PROCESS_SCOPE     = driver-owned; runtime fixtures inherit the Cargo/test process group
```

| Configuration | Compilation/checks                                                   | Test targets                                                                                    |
| ------------- | -------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| Production    | Platform, CLI, and harness with sample features disabled             | Their inline unit targets; production component and service targets                             |
| Development   | Same packages with explicit `dev-sample` features; standalone sample | Their inline unit targets, sample binary unit target, development component and service targets |

Each configuration uses separate invocations for its CLI build and harness tests, with matching feature settings. The
driver builds service artifacts before starting tests; requesting service tests directly without explicit artifact
inputs fails with an actionable message. Rustdoc tests, where present, run with their package's unit-tier command.

All Cargo/tool children owned by the driver have a 15-minute deadline and retained diagnostics; ordinary service
invocations use the shorter per-invocation deadline above. After building, the demonstration replaces the driver with
the built CLI using Unix `exec`, with inherited streams and cwd; it does not create another supervising parent.

```yaml
# .github/workflows/check.yml — engineering:automated-checks
on: [push, pull_request]
permissions:
  contents: read
jobs:
  check:
    strategy:
      fail-fast: false
      matrix:
        os: [ubuntu-24.04, ubuntu-24.04-arm, macos-15, macos-15-intel]
    runs-on: ${{ matrix.os }}
    timeout-minutes: 30
    # Checkout and pinned-tool bootstrap precede the same cargo dev check used locally.
    # No publish credentials, release/tag trigger, Homebrew installation, or running product services.
```

CI host images are development execution environments, not minimum supported release OS/libc promises. Tool/action
versions are pinned in repository configuration; bootstrap and cache setup do not introduce another check list.

| Agent-facing artifact                 | Content contract                                                                                                         |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| Existing repository agent entry point | Routes to the operator-selected canonical context and shared governance through existing workspace discovery             |
| Architecture guidance                 | Actual package/API boundaries, command registration site, error/output ownership, and explicit boundary-change procedure |
| Rust conventions                      | Naming, visibility, dependency changes, typed errors, no runtime panics for user input, unit/component/service placement |
| Command-extension guidance            | Actual parser/facade/main/test touchpoints; no speculative Workspace/Fleet modules                                       |
| Verifiability matrix                  | Available method IDs and exact task commands; commands/manual/tools shape governed by `winter-canon`                     |

| Method ID to expose once implemented | Command                                               |
| ------------------------------------ | ----------------------------------------------------- |
| `winter:check`                       | `cargo dev check`                                     |
| `winter:format-check`                | `cargo dev format`                                    |
| `winter:lint`                        | `cargo dev lint`                                      |
| `winter:architecture`                | `cargo dev architecture`                              |
| `winter:build`                       | `cargo dev build --configuration production`          |
| `winter:unit-test`                   | `cargo dev test unit --configuration production`      |
| `winter:component-test`              | `cargo dev test component --configuration production` |
| `winter:service-test`                | `cargo dev test service --configuration production`   |
| `winter:sample-test`                 | `cargo dev test service --configuration development`  |
| `tool:sample`                        | `cargo dev sample -- inspect`                         |

Matrix entries are added to application context only when their commands exist.
