# Command registration

How a command becomes part of the binary: one file per node of the command tree, discovered at build time, with no
registry a person edits. The tree it must produce is [command-surface.md](./command-surface.md); the package layout is
[repository-layout.md](./repository-layout.md).

## System

```text
# engineering:additive-registration, decision:parallel-workspaces/component-runtime/command-registration
crates/core/src/commands/<leaf>.rs            top-level command          agents.rs        -> `agents`
crates/core/src/commands/<group>/<leaf>.rs    command of a group         ws/index.rs      -> `ws index`
                                                                         service/ext_services.rs -> `service ext-services`
crates/core/build.rs                          walks the directory, writes $OUT_DIR/registry.rs
crates/core/build/discover.rs                 the walk and the code generation, as pure functions build.rs includes
crates/core/src/registry.rs                   include!(concat!(env!("OUT_DIR"), "/registry.rs")); the Entry type
crates/core/src/groups.rs                     the four groups and their descriptions
crates/core/src/parse.rs                      builds the argument parser from REGISTRY and dispatches
```

```text
path rules, enforced by build.rs; a violation fails the build with a message that starts with the offending path
  file stem          [a-z][a-z0-9_]*           command name = the stem with every "_" replaced by "-"
  directory          one of: ext  repo  service  ws      # command-surface.md, Tree
  depth              a file sits in commands/ or in one group directory; nothing deeper
  other entries      no file that is not *.rs; no mod.rs; no directory outside the four groups
  content            the file's text contains the identifier REGISTRATION
rebuild              build.rs prints cargo:rerun-if-changed for crates/core/src (so a commands/ directory that appears
                     later is seen), for commands/ and each group directory when present, and for build/discover.rs
no command           commands/ absent, or present with no file, yields an empty REGISTRY; the build passes.
                     Git tracks no empty directory, so the tree before the first command has no commands/ at all.
```

A command file holds the node's arguments and the adapter from parsed arguments to one call into a domain module. Logic
shared by two commands lives in a domain module, never in `commands/`.

## Interfaces

```rust
// crates/core/src/registry.rs
#[allow(dead_code, reason = "the registration contract: command files are discovered at build time, and a tree with no command or no alias is a supported state")]
pub(crate) enum Registration {
    Command {
        // Receives the node, already named from its file, and returns it with its description, arguments,
        // and options added. Names, arity, and types follow command-surface.md.
        args: fn(clap::Command) -> clap::Command,
        // Runs after parsing succeeded. Writes the result through the Context; returns how the invocation ends.
        run: fn(&clap::ArgMatches, &mut Context<'_>) -> Termination,
        hidden: bool,                      // omitted from help, still dispatched
    },
    // A second name for a sibling in the same group: same arguments, same run function.
    Alias { of: &'static str },            // the sibling's command name, e.g. "up"
}

#[allow(dead_code, reason = "as Registration")]
pub(crate) struct Entry {
    pub path: &'static [&'static str],     // ["ws", "index"]
    pub file: &'static str,                // "crates/core/src/commands/ws/index.rs", for messages
    pub registration: &'static Registration,
}

// Generated. Sorted by `path`, so help order and dispatch do not depend on directory iteration order.
pub(crate) static REGISTRY: &[Entry];
```

The two `allow` attributes are permanent, for the reason [repository-layout.md](./repository-layout.md), What a command
receives, gives; adding the first command removes neither. `parse.rs` reads `REGISTRY` from the start, so the static
itself needs no attribute.

```rust
// a command file, whole shape — crates/core/src/commands/ws/index.rs
use crate::registry::Registration;

pub(crate) const REGISTRATION: Registration = Registration::Command { args, run, hidden: false };

fn args(command: clap::Command) -> clap::Command { /* about, positionals, options */ }
fn run(matches: &clap::ArgMatches, cx: &mut Context<'_>) -> Termination { /* typed params -> domain call */ }

#[cfg(test)]
mod tests { /* unit tests of this file's argument mapping */ }
```

```rust
// an alias file, whole content — crates/core/src/commands/service/start.rs
pub(crate) const REGISTRATION: crate::registry::Registration = crate::registry::Registration::Alias { of: "up" };
```

```rust
// $OUT_DIR/registry.rs — shape only; module identifiers are the generator's own
#[path = "<absolute>/crates/core/src/commands/ws/index.rs"] mod /* … */;
pub(crate) static REGISTRY: &[Entry] = &[
    Entry { path: &["ws", "index"], file: "crates/core/src/commands/ws/index.rs", registration: &/* … */::REGISTRATION },
];
```

```rust
// crates/core/build/discover.rs — included by build.rs and, under cfg(test), by the library, so it is unit-tested
pub struct CommandFile { pub relative: String, pub path: Vec<String> }
pub enum DiscoverError { /* one variant per path rule; each carries the offending relative path */ }
impl std::fmt::Display for DiscoverError { /* the text starts with the offending path, as Entry.file spells one */ }
pub fn classify(relative_paths: &[String], contents: &dyn Fn(&str) -> String) -> Result<Vec<CommandFile>, DiscoverError>;
pub fn generate(files: &[CommandFile], commands_dir_absolute: &str) -> String;
```

On a `DiscoverError`, `build.rs` prints its `Display` text as the first line of its error output and exits non-zero; it
adds no prefix of its own before the path.

### Parser construction

```text
# crates/core/src/parse.rs — builder interface; the tree is assembled from a slice of Entry, so a test can pass its own
root        clap::Command named Identity.program; long --help and long --version only (no -h, no -V);
            no `help` subcommand; no prefix matching of commands or long options
groups      one subcommand per group that has at least one Entry; description from groups.rs;
            a group with no Entry does not exist: it is absent from help and is an unknown command
leaves      Registration::Command -> args(clap::Command::new(<name>)); hidden as declared
            Registration::Alias   -> the sibling's args under the alias name; dispatch calls the sibling's run
consistency checked by a unit test over REGISTRY, which fails naming Entry.file:
              an Alias whose `of` names no Command in the same group, or names another Alias
              a top-level leaf whose name equals a group name
              a clap debug assertion raised while building any node
```

```text
# what the foundation's parser answers; every other root behaviour of results-and-exit-codes.md, Flows, is added
# with the rest of this epic
--version                 stdout: "<program> <version>" on one line         Termination::Success
--help                    stdout: root help                                 Termination::Success
<group> --help, <command> --help      stdout: that node's help              Termination::Success
no arguments              nothing written       Termination::BareGroup { help: the root help }             exit 2
bare group                nothing written       Termination::BareGroup { help: that group's help }         exit 2
unknown command or option, missing or malformed argument                    Termination::Usage, exit 2
a registered leaf         run(matches, cx)                                  its Termination
```

A bare root and a bare group end the same way, as [results-and-exit-codes.md](./results-and-exit-codes.md), Domain and
Error rendering, fix it: `run` writes nothing to either handle and returns `Termination::BareGroup`, whose `help` is the
text `--help` would have written to stdout for that node. `diagnostic()` returns that text, and the binary writes it to
stderr as it writes every diagnostic. `Termination::Usage` carries `UsageError::Parse` with the command path parsed so
far and the `ParseFault` of the same Domain section.

### Formatting discovered files

```text
`cargo fmt` follows `mod` declarations from each target's root and so does not reach a file declared only in
generated code, nor one a build script includes textually. The format step of gates.md passes every
crates/core/src/commands/**/*.rs path, and crates/core/build/discover.rs, to rustfmt explicitly, in check mode and in
write mode alike. With no commands/ directory the list is discover.rs alone.
```

### Exercising registration by hand

```text
That the build picks up a new command file, and that the build script itself stops the build, cannot be tests of the
tree they change. Both are exercised by hand, in a scratch copy of the checkout under a temporary directory (never in
the checkout itself), once per case:
  add crates/core/src/commands/ws/probe.rs: REGISTRATION is a Registration::Command whose `run` returns
  Termination::Success
      cargo build --workspace --locked exits 0; `winter-workspace ws probe` exits 0; `winter-workspace --help`
      lists `ws`; cargo clippy --locked --workspace --all-targets --all-features -- -D warnings exits 0
  rename that file to crates/core/src/commands/ws/Bad.rs
      cargo build --workspace --locked exits non-zero, and the build script's first error line starts with
      crates/core/src/commands/ws/Bad.rs
Success: each run ends as stated, and `git status` of the scratch copy shows only the one added file.
Everything else about discovery is automatic: the path rules, the Display text, and the generated registry are
unit-tested through `classify` and `generate`.
```

## Flows

```text
add a command
  1. write crates/core/src/commands/<group>/<name>.rs with REGISTRATION
  2. write its tests: unit tests in the file; crates/core/tests/<group>_<name>.rs for behaviour in process;
     scenarios/<group>-<name>/<scenario>/ for recorded behaviour
  3. build: build.rs regenerates the registry; the command is in help and dispatch
no other tracked file changes, unless the command needs something repository-layout.md lists as a shared edit
```
