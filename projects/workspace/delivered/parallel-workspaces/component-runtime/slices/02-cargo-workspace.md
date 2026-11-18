---
name: cargo-workspace
depends-on:
  - wipe-python-tree
---

# Cargo workspace, four packages, and the binary

Creates the Cargo workspace of [repository-layout.md](../spec/repository-layout.md) — Cargo workspace, Packages and
edges, Domain, The binary — under `decision:parallel-workspaces/component-runtime/crate-layout`, delivering the
toolchain and lock part of `engineering:reproducible-build` and the boundary of `engineering:library-output-boundary`
and `engineering:composition-root`.

## Change

- `rust-toolchain.toml`, the root `Cargo.toml`, and a committed `Cargo.lock` exist as the spec gives them.
- All four packages exist, so no later change edits the member list: `crates/cli` with `src/main.rs`; `crates/core` with
  the facade `src/lib.rs`, `src/process.rs`, `src/termination.rs`, `src/context.rs`; `crates/test-support` with a
  `src/lib.rs` holding only a crate doc comment; `crates/dev` as the spec's tree and the paragraph under it give a
  driver package with no command: `src/lib.rs` exposing `main_with` with the signature of [gates.md](../spec/gates.md),
  Steps, which returns 2 for every argument vector, and the thin `src/main.rs`.
- `termination.rs` defines exactly the types and variants of the spec's Errors the foundation defines, each spelled as
  [results-and-exit-codes.md](../spec/results-and-exit-codes.md), Domain, and
  [configuration-model.md](../../../../work/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
  Domain, spell it, with `Termination::exit_code` and `Termination::diagnostic`. `Termination::BareGroup { help }` is
  among them: exit code 2, and its diagnostic is its `help` text. `InternalError` derives `Debug` only and implements
  `Display` and `std::error::Error` by hand, as the same section of the layout spec gives them; every other error type
  derives `thiserror::Error`. `crates/core` depends on `thiserror` and `toml` and on no other third-party crate.
- `context.rs` defines `Context` with the spec's `allow` attribute and every field of the spec except `files`, which
  arrives with the first command that reads a file. `run` builds a `Context` from the `Invocation`.
- `process.rs` defines `ProcessEnv` with the spec's constructor `from_vars` and `get`.
- `winter_workspace_core::run` answers `--version` through the `Context`'s `out` handle and returns `Termination::Usage`
  carrying `UsageError::Parse`, with an empty command path and the `ParseFault` the Domain section gives for an
  unrecognised argument, for every other argument vector. `main.rs` does the five steps of the spec and nothing else.
- `AGENTS.md` gains the standing rule about the library and `crates/cli/src/main.rs`.
- Files touched: `Cargo.toml`, `Cargo.lock`, `rust-toolchain.toml`, `AGENTS.md`, `crates/cli/**`, `crates/core/**`,
  `crates/test-support/**`, `crates/dev/**`.
- No gate driver exists yet: the change is admitted by `cargo build --workspace --locked` and
  `cargo test --workspace --locked`, and its tree passes the format and Clippy commands of [gates.md](../spec/gates.md),
  Which check admits a change.

## Acceptance

- `cargo build --workspace --locked` succeeds on toolchain 1.99.1 and produces `target/debug/winter-workspace`;
  `git status` is clean afterwards.
- `cargo fmt --all --check`, `cargo clippy --locked --workspace --all-targets --all-features -- -D warnings`, and
  `cargo test --locked --workspace` exit 0.
- The only lint attribute in `crates/` is the `allow` on `Context`; no `#[expect]` occurs.
- A unit test calls `run` with `["--version"]` and in-memory handles and reads `winter-workspace 0.1.0` and a newline
  from the stdout buffer, an empty stderr buffer, and `Termination::Success`.
- A unit test calls `run` with `["--nope"]` and reads `Termination::Usage` and empty buffers.
- A unit test over `Termination` asserts each variant's exit code against the Domain block of
  [results-and-exit-codes.md](../spec/results-and-exit-codes.md) — `Reported` is 1, and `Usage` and `BareGroup` are 2 —
  that `Success` and `Reported` have no diagnostic, that the diagnostic of `BareGroup { help }` equals `help`, and that
  the diagnostic of a `Config` termination holding `ConfigError::RootNotFound` names its `start` path.
- A unit test builds an `InternalError::Unexpected` over an `std::io::Error` and reads that error's text from `Display`
  and `Some` from `source()`; an `InternalError::Panic` gives its message and `None`.
- A unit test builds a `ProcessEnv` with `from_vars` from two pairs and reads one back with `get` and `None` for an
  absent key.
- `crates/core/src` contains no `mod.rs` and none of the spec's forbidden tokens.
- `target/debug/winter-workspace-dev` exits 2 when run with any argument, and a unit test of `main_with` with an
  environment lookup that returns nothing reads 2.

## Out of scope

- Argument parsing, help, and command dispatch: `command-registration`.
- The gate commands and Cargo aliases: `gates`.
- Any content of the test-support package: `process-harness`.
- The file port: it arrives with the first command that reads a file.
