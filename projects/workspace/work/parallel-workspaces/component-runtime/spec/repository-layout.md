# Repository layout

The tracked tree of the `winter-workspace` code repository on its Rust branch: what the wipe removes and rewrites, the
Cargo workspace, the packages and their allowed edges, and the boundary between the library and the binary. Command
discovery is [command-registration.md](./command-registration.md); the gate commands are [gates.md](./gates.md); test
support is [test-harness.md](./test-harness.md); the context tree is [contributor-context.md](./contributor-context.md).
The termination taxonomy this layout carries is [results-and-exit-codes.md](./results-and-exit-codes.md).

## System

### Root after the wipe

```text
# engineering:rust-only-branch, decision:parallel-workspaces/component-runtime/retained-root-documents
# decision:parallel-workspaces/component-runtime/ci-workflow
removed, whole trees:   tools/        context/        skills/
removed, file:          .github/workflows/ci.yml      # its two jobs lint the removed Markdown and test the removed Python
kept, byte for byte:    LICENSE       PRINCIPLES.md
rewritten:              README.md  AGENTS.md  CLAUDE.md  .gitignore  dprint.json  .rumdl.toml
tracked set afterwards: exactly the eight files of the last two rows
```

| File          | Content after the rewrite                                                                                                                                                                                                                    |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `README.md`   | What the repository is: the Rust source of the `winter-workspace` binary. How to build it. A pointer to `AGENTS.md` for contributors and to `PRINCIPLES.md`. The licence paragraph. No feature tour, no install steps, no command reference. |
| `AGENTS.md`   | The one-sentence description and the standing rules of [contributor-context.md](./contributor-context.md), Entry point, that are true of the tree at that moment. Until the context tree exists it has no routing table.                     |
| `CLAUDE.md`   | A `# winter-workspace` title and the line `@AGENTS.md`.                                                                                                                                                                                      |
| `.gitignore`  | `target/` only.                                                                                                                                                                                                                              |
| `dprint.json` | Markdown wrapped at 120 columns, asterisk emphasis, dash lists; the plugins `markdown-0.29.0`, `json-0.26.0`, `toml-0.10.0`, by their `https://plugins.dprint.dev/<name>.wasm` addresses; excludes `target/**` and `scenarios/**`.           |
| `.rumdl.toml` | `MD018` disabled; `MD004` style `dash`; `MD013` line length 120 with tables and code blocks exempt.                                                                                                                                          |

`dprint.json` and `.rumdl.toml` configure a contributor's editor and hand-run tools. No gate runs either tool, under
`decision:parallel-workspaces/component-runtime/gate-scope`.

### Tracked tree once the foundation is built

```text
.cargo/config.toml            # aliases; gates.md
.github/workflows/check.yml   # gates.md
Cargo.toml  Cargo.lock  rust-toolchain.toml
AGENTS.md  CLAUDE.md  README.md  LICENSE  PRINCIPLES.md  .gitignore  dprint.json  .rumdl.toml
context/                      # contributor-context.md
crates/
  cli/                        # package winter-workspace            bin winter-workspace
    Cargo.toml
    src/main.rs               # the only file that names std::io::stdout, stderr, std::env, std::process
    tests/<subject>.rs        # executable tests, one file per subject
    tests/scenarios.rs        # the recorded-scenario target; harness = false
  core/                       # package winter-workspace-core       lib
    Cargo.toml
    build.rs                  # command-registration.md
    build/discover.rs
    src/lib.rs                # facade
    src/process.rs            # run, Invocation, Identity, ProcessEnv
    src/termination.rs        # Termination and the error enums it carries
    src/context.rs            # Context
    src/files.rs              # the file port and its one adapter
    src/registry.rs  src/groups.rs  src/parse.rs      # command-registration.md
    src/commands/             # one file per command; absent until the first command exists
    src/<domain>.rs, src/<domain>/…
    tests/<subject>.rs        # component tests, one file per subject
  test-support/               # package winter-workspace-test-support   lib, publish = false
  dev/                        # package winter-workspace-dev            lib + bin winter-workspace-dev, publish = false
    src/lib.rs                # every module of the driver; unit-tested here; its one entry is main_with (gates.md, Steps)
    src/main.rs               # calls main_with with the process arguments, std::env::var_os as the environment
                              # lookup, and the repository root, and exits with its code; nothing else
    tests/<subject>.rs        # integration tests of the driver; they import the library target
scenarios/                    # recorded scenarios; owned by epic:rust-adoption/parity-validation
  reference.toml
  <command path joined by "-">/<name>/        # ws-index/alias-suggested/
```

`main_with` has the signature of [gates.md](./gates.md), Steps, in every tree that has the driver package, and answers 2
to arguments that name no command the driver has. A driver package with no command is therefore complete: its library
holds `main_with` returning 2, and its `main.rs` never changes when a command is added.

### Cargo workspace

```toml
# Cargo.toml — engineering:reproducible-build, decision:parallel-workspaces/component-runtime/crate-layout
[workspace]
members = ["crates/cli", "crates/core", "crates/dev", "crates/test-support"]
default-members = ["crates/cli", "crates/core"]
resolver = "3"

[workspace.package]
version = "0.1.0"
edition = "2024"
rust-version = "1.99"
license = "FSL-1.1-ALv2"

[workspace.dependencies]
winter-workspace-core = { path = "crates/core" }
winter-workspace-test-support = { path = "crates/test-support" }

[profile.dev]
panic = "unwind"
[profile.release]
panic = "unwind"
```

```toml
# rust-toolchain.toml
[toolchain]
channel = "1.99.1"
profile = "minimal"
components = ["rustfmt", "clippy"]
```

`default-members` makes a bare `cargo build` or `cargo run` produce the product alone. Every check names the whole
workspace explicitly (`--workspace`, or `-p`), so no check depends on the default.

Third-party versions are declared in the manifest of the package that uses them and resolved in the committed
`Cargo.lock`. No `rustfmt.toml`, `clippy.toml`, or `[workspace.lints]` exists: rustfmt and Clippy run with their
defaults, and warnings are denied by the gate.

A manifest names a third-party dependency only while a source file of its package uses it. The table gives each
dependency with what uses it.

| Package                         | Third-party dependencies, each with its user in parentheses                                                                                                                                                                                                                    |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `winter-workspace`              | none                                                                                                                                                                                                                                                                           |
| `winter-workspace-core`         | `thiserror` 2 and `toml` (error types); `clap` 4.6 with `default-features = false` and `std`, `help`, `usage`, `error-context`, `suggestions` (parser); `sha1` (env index) and `serde_json` (first command with `--json`); `toml_edit` and `libc` (first configuration writer) |
| `winter-workspace-test-support` | `libc`, `tempfile`, `serde`, `serde_json`, `toml`, `sha1`                                                                                                                                                                                                                      |
| `winter-workspace-dev`          | `cargo_metadata` and `toml` (architecture rules: the package graph, and the alias table of `.cargo/config.toml`), `libc` (step process groups)                                                                                                                                 |

`serde_json` is the library's one JSON encoder: every `--json` document and event is a `serde_json::Value` written with
`serde_json::to_writer`, followed by a newline. No module escapes a JSON string by hand.

### Packages and edges

| Package                         | Role                                                                                | May depend on (normal)          | May depend on (dev)             |
| ------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------- | ------------------------------- |
| `winter-workspace`              | Composition root: binds streams, environment, cwd; maps the termination to an exit. | `winter-workspace-core`         | `winter-workspace-test-support` |
| `winter-workspace-core`         | Parsing, commands, domain, adapters. Returns values.                                | —                               | `winter-workspace-test-support` |
| `winter-workspace-test-support` | Isolated trees, process runner, git and workspace fixtures, scenario machinery.     | —                               | —                               |
| `winter-workspace-dev`          | Gate driver, formatter driver, architecture rules, scenario recorder.               | `winter-workspace-test-support` | —                               |

```text
# engineering:testable-boundaries — crates/dev/tests/architecture.rs, over `cargo metadata --no-deps`
# the rules are functions of crates/dev/src/architecture.rs, a module of the driver's library target
member set     exactly the four packages above; an extra or missing member fails
edges          every workspace edge present is in the table, with its kind; any other edge fails
build edges    none between workspace packages
publish        test-support and dev declare publish = false
```

## Domain

```rust
// crates/core/src/lib.rs — the facade. Modules are private; the public surface is these named re-exports.
// engineering:library-output-boundary, engineering:composition-root
pub use process::{run, Identity, Invocation, ProcessEnv};
pub use termination::{
    CommandError, ConfigError, FileCause, InternalError, ParseFault, Termination, TomlParseError, UsageError,
};

pub struct Identity {
    pub program: &'static str,     // "winter-workspace"; never derived from argv[0]
    pub version: &'static str,     // the binary package's CARGO_PKG_VERSION, passed in by main
}

pub struct ProcessEnv(/* a snapshot of the process environment taken once by main */);
impl ProcessEnv {
    // The one constructor. main passes std::env::vars_os(); a test passes the pairs it wants.
    pub fn from_vars(vars: impl IntoIterator<Item = (std::ffi::OsString, std::ffi::OsString)>) -> Self;
    pub fn get(&self, key: &str) -> Option<&std::ffi::OsStr>;
}

pub struct Invocation<'a> {
    pub args: Vec<std::ffi::OsString>,          // argv without argv[0]
    pub cwd: std::path::PathBuf,
    pub env: ProcessEnv,
    pub identity: Identity,
    pub stdin: &'a mut dyn std::io::BufRead,
    pub stdout: &'a mut dyn std::io::Write,
    pub stderr: &'a mut dyn std::io::Write,
    pub stdout_is_terminal: bool,
    pub stderr_is_terminal: bool,
}

// Parses, dispatches, and returns. Writes results, the help a --help asks for, and reporter lines to the handles
// of `invocation`. Never writes the final diagnostic and never exits. Builds the Context, binding OsFiles to its
// file port.
pub fn run(invocation: Invocation<'_>) -> Termination;

impl Termination {
    pub fn exit_code(&self) -> u8;                  // results-and-exit-codes.md, Exit codes
    pub fn diagnostic(&self) -> Option<String>;     // None for Success, Reported; the `help` text for BareGroup
}
```

### Errors the foundation defines

```text
# crates/core/src/termination.rs — engineering:typed-outcomes
# every type is spelled as its owning spec spells it; a type not listed here does not exist yet
Termination      Success  Reported  Usage  BareGroup  Command  Config  Internal results-and-exit-codes.md, Domain
UsageError       Parse { command_path, fault }                                 same; Rule arrives with the first command
ParseFault       all six variants                                              rule classed as usage
CommandError     the enum, with no variant                                     same; a variant arrives with its module
ConfigError      RootNotFound { start }                                        ../../workspace-configuration/spec/
                 FileRead { path, cause }                                        configuration-model.md, Domain
                 EnvsPerWorkspaceTooSmall { envs_per_workspace, aliases }
FileCause        Io  Toml                                                      same
TomlParseError   wraps toml::de::Error                                         same
InternalError    Unexpected  Panic                                             results-and-exit-codes.md, Domain
```

Each remaining `Termination` variant, and each remaining variant of an enum above, is added by the behaviour that first
returns it. All of these are public through the facade, so none needs a constructor in the library to compile without
warnings.

```rust
// InternalError is the one error type that does not derive thiserror::Error. Its variants hold a
// std::backtrace::Backtrace, and thiserror's derive, for any field of that type, generates a `provide` method over
// the standard library's error-request API (core::error::Request). That API has been unstable, and the layout
// does not depend on whether toolchain 1.99.1 offers it: the type derives Debug and implements the two traits by
// hand, which compiles on every stable toolchain.
impl std::fmt::Display for InternalError { /* Unexpected: the source's Display · Panic: the message */ }
impl std::error::Error for InternalError {
    fn source(&self) -> Option<&(dyn std::error::Error + 'static)> { /* Unexpected: Some(its source) · Panic: None */ }
    // no `provide`; the boundary reads the backtrace from the variant's public field
}
// No other error type of the library holds a Backtrace field, so every other one derives thiserror::Error.
```

### What a command receives

```rust
// crates/core/src/context.rs — built by `run` from the Invocation.
#[allow(dead_code, reason = "the command contract: a field stays whether or not a present command reads it")]
pub(crate) struct Context<'a> {
    pub cwd: &'a std::path::Path,
    pub env: &'a ProcessEnv,
    pub identity: &'a Identity,
    pub stdin: &'a mut dyn std::io::BufRead,
    pub out: &'a mut dyn std::io::Write,
    pub err: &'a mut dyn std::io::Write,
    pub out_is_terminal: bool,
    pub err_is_terminal: bool,
    pub files: &'a dyn ReadFiles,
    // pub file_writer: &'a dyn WriteFiles — added, with OsFiles's implementation, by the first command that writes
}
```

```rust
// crates/core/src/files.rs — engineering:domain-io-boundaries. The only source file of the library that names
// std::fs. Reading and writing are separate authorities: a module that takes only ReadFiles cannot change a file.
pub(crate) trait ReadFiles {
    fn read(&self, path: &std::path::Path) -> std::io::Result<Option<Vec<u8>>>;   // Ok(None): no such file
    fn is_file(&self, path: &std::path::Path) -> bool;                            // a regular file, links followed
}
pub(crate) trait WriteFiles {
    // One read-modify-write of one file with the lock and replacement rules of
    // ../../workspace-configuration/spec/configuration-model.md, Write integrity.
    // `edit` gets the current bytes, None when the file is absent, and returns the new bytes,
    // or None to leave the file as it is. Returns whether the file was replaced.
    fn update(
        &self,
        path: &std::path::Path,
        edit: &mut dyn FnMut(Option<&[u8]>) -> Option<Vec<u8>>,
    ) -> std::io::Result<bool>;
}
pub(crate) struct OsFiles;      // implements ReadFiles from the start, WriteFiles with the first writer
```

```text
warning-free while partly used
  Context, and Registration and Entry of command-registration.md, are contracts that command files fill. Command files
  are discovered at build time, so a tree with no command, or with no alias, constructs and reads only part of them and
  must still pass a gate that denies warnings. Each of the three carries #[allow(dead_code, reason = "…")] permanently.
  #[expect(dead_code)] is never used on them: the first command would turn the expectation into a failure.
  No other item of the library carries a lint attribute for code that nothing uses; such code is not written yet.
git
  No command of the foundation calls git. The port for git calls is specified with the first behaviour that makes one.
```

## Interfaces

### The binary

```rust
// crates/cli/src/main.rs — whole responsibility
fn main() -> std::process::ExitCode {
    // 1. snapshot argv (without argv[0]), cwd, environment: ProcessEnv::from_vars(std::env::vars_os())
    // 2. lock stdin, stdout, stderr; ask each output whether it is a terminal
    // 3. termination = winter_workspace_core::run(Invocation { identity: Identity {
    //        program: "winter-workspace", version: env!("CARGO_PKG_VERSION") }, .. })
    // 4. flush stdout; write termination.diagnostic(), when Some, to stderr, with a newline added when the text
    //    does not end in one
    // 5. ExitCode::from(termination.exit_code())
}
```

A failed write to stdout or stderr in step 4 ends the process with exit 1 and writes nothing further.

### Library rules

```text
# engineering:library-output-boundary — checked by crates/dev/tests/architecture.rs over crates/core/src/**
forbidden tokens   std::io::stdout   std::io::stderr   io::stdout(   io::stderr(   println!   eprintln!   print!
                   eprint!   dbg!   std::process::exit   process::exit(   std::env::var   std::env::vars
                   std::env::args   std::env::current_dir
                   std::fs — everywhere except crates/core/src/files.rs
exempt             nothing else; a test module inside the library is checked like the rest
modules            no mod.rs; a module `x` is `x.rs`, with children in `x/`
errors             thiserror enums or structs that carry the values their message names — InternalError alone
                   implements the traits by hand (Errors the foundation defines); no String errors across a
                   module boundary; no unwrap, expect, or panic! on a value that comes from the user, a file, or a
                   child process
unsafe             only with a comment that states why it is sound
```

The token and `mod.rs` rules are checked by the architecture test. The error and `unsafe` rules are checked in review.

### What a later change may add without editing a tracked file

| Adds                    | Where                                                       | Discovered by                                        |
| ----------------------- | ----------------------------------------------------------- | ---------------------------------------------------- |
| A command               | `crates/core/src/commands/…`                                | [command-registration.md](./command-registration.md) |
| A component test file   | `crates/core/tests/<subject>.rs`                            | Cargo test-target discovery                          |
| An executable test file | `crates/cli/tests/<subject>.rs`                             | Cargo test-target discovery                          |
| A recorded scenario     | `scenarios/<command path joined by "-">/<name>/`            | the scenario runner's directory walk                 |
| A context document      | `context/<area>/<subject>.md`, linked from the area's index | — (the index is edited)                              |

### What stays a shared edit

```text
a new domain module          one `mod` line in crates/core/src/lib.rs, or in the parent module file
a new Termination variant    crates/core/src/termination.rs
a new CommandError variant   crates/core/src/termination.rs      (the module's own error enum lives in the module,
                             and crates/core/src/lib.rs re-exports it with the types its variants carry)
a new ConfigError variant    crates/core/src/termination.rs
a new port, or a new field of Context     crates/core/src/context.rs and crates/core/src/process.rs
a new third-party crate      the package's Cargo.toml and Cargo.lock
```

A body of work that is built as parallel changes lands these edits in one change first, with the module files it
declares present and empty, and the parallel changes then fill files they alone own.
