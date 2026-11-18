---
name: command-registration
depends-on:
  - cargo-workspace
---

# Command discovery by file location, and the parser built from it

Builds [command-registration.md](../spec/command-registration.md) whole: the build script, the registry types, the
groups, and the parser with the foundation's answers. Delivers `engineering:additive-registration` under
`decision:parallel-workspaces/component-runtime/command-registration`, with help placement under
`decision:parallel-workspaces/component-runtime/help-rendering`.

## Change

- `crates/core/build.rs` and `build/discover.rs` walk `src/commands/`, enforce the path rules, and write the registry to
  `OUT_DIR`; `src/registry.rs` includes it and defines `Registration` and `Entry` with the spec's two `allow`
  attributes.
- `src/groups.rs` holds the four groups; `src/parse.rs` builds the parser from a slice of `Entry` and dispatches; `run`
  uses it over `REGISTRY` and passes the `Context` to the dispatched command.
- No command file exists and `src/commands/` is absent; the spec's "no command" rule covers both the absent and the
  empty directory.
- The binary answers `--version`, `--help`, a bare invocation, and unknown input as the spec's table says, and lists no
  group. A bare root or group returns `Termination::BareGroup` carrying that node's help; `run` writes nothing for it,
  and the binary writes the help to stderr as the termination's diagnostic.
- `clap` joins `crates/core/Cargo.toml` with the version and features
  [repository-layout.md](../spec/repository-layout.md), Cargo workspace, names.
- `crates/core/tests/root.rs` is the first component test: it calls `run` with in-memory handles.
- `AGENTS.md` gains the standing rule about command files.
- Files touched: `crates/core/build.rs`, `crates/core/build/discover.rs`, `crates/core/Cargo.toml`, `Cargo.lock`,
  `crates/core/src/{lib,process,registry,groups,parse}.rs`, `crates/core/tests/root.rs`, `AGENTS.md`.
- No gate driver exists yet: the change is admitted and kept lint-clean as [gates.md](../spec/gates.md), Which check
  admits a change, says for such a tree.

## Acceptance

- `cargo build --workspace --locked`, `cargo test --workspace --locked`,
  `cargo fmt --all --check -- crates/core/build/discover.rs`, and
  `cargo clippy --locked --workspace --all-targets --all-features -- -D warnings` exit 0, with no `src/commands/`
  directory and again with an empty one.
- The lint attributes in `crates/` are the three `allow`s the specs give `Context`, `Registration`, and `Entry`; no
  `#[expect]` occurs.
- Unit tests of `classify` return one `DiscoverError` per path rule, each carrying the offending path: an upper-case
  stem, a file two directories deep, a directory outside the four groups, a `mod.rs`, a non-`.rs` file, a file without
  `REGISTRATION`. For each, the `Display` text starts with that path.
- A unit test of `generate` for `ws/index.rs` and `agents.rs` yields entries with paths `["agents"]` and
  `["ws", "index"]` in that order, and `service/ext_services.rs` yields the name `ext-services`.
- The two runs of the spec's Exercising registration by hand, made in a temporary copy of the checkout, each end as
  stated there: the added `ws/probe.rs` dispatches and puts `ws` in root help with no other file changed and the Clippy
  command still exiting 0; the same file named `Bad.rs` fails the build with a first error line that starts with its
  path. This is the manual method for the build script's own failure; nothing automatic runs a failing build.
- A unit test builds the parser from a test-owned slice holding a command, a hidden command, and an alias, and asserts:
  the command dispatches to its `run`; the alias dispatches to the same `run`; the hidden command is absent from help
  text and dispatches; a group without entries is an unknown command; the bare group returns `Termination::BareGroup`
  whose `help` names the command and the alias and not the hidden command, with nothing written to either handle.
- A unit test over a slice with an alias that names no sibling fails the consistency check and names the alias file; the
  same check passes over `REGISTRY`.
- `crates/core/tests/root.rs`: `--help` writes to stdout with `Success`; no arguments writes nothing to stdout and
  nothing to stderr and returns `Termination::BareGroup` whose `help` equals the text `--help` wrote to stdout, with
  exit code 2 and that text as its diagnostic; `--nope` returns `Termination::Usage`; `-h` and `-V` are usage errors; an
  abbreviated long option is a usage error; `--version` still answers as before.
- Run by hand, `target/debug/winter-workspace` with no arguments exits 2 with empty stdout and the root help on stderr.

## Out of scope

- Any command: each arrives as its own file with the epic that owns it.
- Root options, logging, and the remaining root behaviour of
  [results-and-exit-codes.md](../spec/results-and-exit-codes.md): later slices of this epic.
