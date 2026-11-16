# Contributor context

The agent-facing documents of the code repository: the entry point, the context tree, and the verifiability matrix. They
describe how to build the Rust tool and nothing else. The tree they describe is
[repository-layout.md](./repository-layout.md); the commands they name are [gates.md](./gates.md).

The `context` command of [command-surface.md](./command-surface.md), which serves installed context to a workspace's
agents, is a product command and no part of this tree.

## System

```text
# engineering:workspace-development-context
AGENTS.md                         entry point: standing rules and the routing table
CLAUDE.md                         "@AGENTS.md"
context/
  index.md                        routes to the four areas
  architecture/
    index.md
    structure.md                  packages, allowed edges, the library and binary boundary, module rules
    changes.md                    where a change goes: a command, a domain module, an error, a dependency
  standards/
    index.md
    rust.md                       the Library rules of repository-layout.md and its "warning-free while partly used"
                                  block: error types, output and environment access, module files, unsafe,
                                  lint attributes
    testing.md                    the test tiers, what each proves, the isolated tree, what a test never does
  tooling/
    index.md
    development.md                bootstrap, the gate commands, formatting, what the driver prints and exits with,
                                  where a build puts the binary
  verification/
    index.md
    matrix.md                     the verifiability matrix
    scenarios.md                  recorded scenarios; added by epic:rust-adoption/parity-validation
```

```text
rules for every document in the tree
  one owner per fact     a fact is stated in one document; others link to it
  reachable              every document is linked from its area index; every index from context/index.md;
                         context/index.md from AGENTS.md
  links resolve          every relative link names an existing tracked file
  commands are real      every command line in a code block is one of gates.md, Commands, or a plain cargo or git call
  scope                  a document may say "the reference tool"; it names neither that tool's language, nor its
                         tooling, nor a path the wipe removed. Mechanically: no scope token below occurs in it.
  no history             no document says what the repository used to contain

scope tokens — the one list; matched as substrings, ASCII case-insensitively
  python   uv run   uv sync   pytest   pip install   tools/winter-cli   .venv   skills/

checked by crates/dev/tests/architecture.rs
  reachable, links resolve     over AGENTS.md and context/**/*.md
  scope                        over AGENTS.md, README.md, and context/**/*.md, the standing rules included;
                               context/verification/scenarios.md alone is exempt
```

```text
facts with one owner in context/standards/testing.md; every other document links to it and restates none
  signals          what a test sends to another process, in its whole extent, as test-harness.md, invoke, signals
                   sent, gives it: the runner's SIGKILL of the group of a child it started, the one test of the
                   support package that ends a process that left that group, and nothing else — no SIGINT and no
                   other signal to a subject. The same rule is a bullet under "What a test never does".
                   context/verification/matrix.md links to it where it says why the interrupt is verified by hand.
                   The driver is no subject of the rule: its step runner kills the group of a step that ran past
                   its deadline, which is the tooling and no test of the product.
  shared helpers   where a helper that more than one test file needs lives, as test-harness.md, System, gives it:
                   the support package for one that reads or builds a tree, the scaffold's `Ended` for an
                   assertion on what `run` answered. The test file of a new command copies neither from another.
facts with one owner in context/architecture/structure.md
  the clock        which sources of the binary read it, as tracing.md, Start instant, gives it: main.rs and
                   trace.rs, and no source of the library
```

## Interfaces

### Entry point

```text
AGENTS.md
  1. what the repository is, in one sentence
  2. standing rules — the rules a session must hold without opening another file:
       run `cargo dev gate slice` before handing a change over; run `cargo dev gate full` before a merge
       a command is one file under crates/core/src/commands/; no registry is edited
       the library returns values; only the binary package touches process streams, environment, sockets, and exit
       tests of the library, the binary, and the support package run in an isolated tree from
         winter-workspace-test-support; such a test never touches a real workspace, a real repository, the network,
         or anything else outside its tree; the only host programs it runs are git and the base system utilities on
         the pinned PATH, and nothing of the developer's own environment reaches it
         two exceptions, and no other: a test may read, and never write, the committed scenarios/ directory; and a
         test may open sockets on the loopback address alone: the listener of the support package's trace collector
         in its own process, and the sockets context/standards/testing.md lists beside it. No test resolves a name
         or reaches another address.
         the driver's own tests in crates/dev/ test the tooling: those of the gates and of the architecture rules
         read the checkout and start Cargo, and those of the record command run in an isolated tree that holds a
         stand-in checkout, and start git and a stand-in executable there
       nothing in the repository starts the reference tool; `cargo dev record` alone is handed its path
       the built binary, however it is started outside the tests and the gates, `cargo run` included, runs only
         with its working directory inside an isolated tree, a scratch workspace under a temporary directory, or
         a scratch copy of the checkout under a temporary directory, and only where no directory at or above the
         working directory holds a `.winter/config.toml` other than one made for the run; never in the checkout
         and never in a real workspace, because it resolves the workspace upward from its working directory and
         `repo remove` writes to the configuration it finds there
         it is never installed, and never put on PATH under the name of the installed `winter`
         the isolated tree is the one context/standards/testing.md describes; a scratch workspace is a directory
         made for the run that holds a `.winter/config.toml` of its own
  3. routing table — one row per area index, each with "read when"
a rule is present only while the tree makes it true: the gate rule from the change that adds the gates, the command
rule from the change that adds registration, the test rule from the change that adds the harness, the reference-tool
rule from the change that adds `cargo dev record`, the built-binary rule once the binary has a command, the loopback
exception from the change that adds the trace collector of test-harness.md, and the word "sockets" in the
library-boundary rule from the change that makes the binary open one
context/standards/testing.md owns which sockets a test opens, as test-harness.md, Collector, lists them: the entry
point says loopback alone, names the collector's listener, and points there for the rest, restating no other item
context/standards/testing.md owns likewise where a test builds the real trace exporter and which trust store that
exporter reads, as tracing.md, Test seams, words both: only in a child started under the pinned environment, with
SSL_CERT_FILE naming a certificate file the test wrote in its tree. The entry point gains no third exception for
it: such a test reads nothing of the host's trust store, so the rule that a test touches nothing outside its tree
holds for it as written
the clause on the driver's own tests is the sentence the checkout's AGENTS.md carries today: those tests are no tests
of the product, they read the checkout because the checkout is their subject, and the isolated-tree rule does not
bind them
the built-binary rule is worded by what is started, not by how: it binds `cargo run`, a path typed in a shell, and a
script alike, and it does not bind a test or a gate, which the test rule binds
AGENTS.md owns the built-binary rule: context/tooling/development.md says where a build puts the binary and that it
is started by its path, links to the rule, and does not restate it; no other document names `cargo run` as the way
to start the binary
```

### Routing

| Document                            | Read when                                                                              |
| ----------------------------------- | -------------------------------------------------------------------------------------- |
| `context/architecture/structure.md` | You add a module or a dependency, or need to know what may depend on what.             |
| `context/architecture/changes.md`   | You add a command, an error kind, or a domain module and need the files it touches.    |
| `context/standards/rust.md`         | You write or review Rust in the library or the binary.                                 |
| `context/standards/testing.md`      | You write a test or choose the tier a behaviour is tested at.                          |
| `context/tooling/development.md`    | You set up a checkout, run a gate, or read a gate failure.                             |
| `context/verification/matrix.md`    | You state how a change is verified, or check that a change's verification is complete. |
| `context/verification/scenarios.md` | You add, record, replay, or debug a recorded scenario.                                 |

### Verifiability matrix

`context/verification/matrix.md` has the three parts and the id forms of `canon:verifiability-matrix`, which it names as
the owner of its shape. Its scope is `workspace`. The content of each part:

Commands.

| Method                      | Command                                          | Proves                                                                          | Use when                                                   |
| --------------------------- | ------------------------------------------------ | ------------------------------------------------------------------------------- | ---------------------------------------------------------- |
| `workspace:unit-test`       | `cargo test -p winter-workspace-core --lib`      | A pure rule: parsing, derivation, ordering, classification.                     | The rule has no file, git, or process input.               |
| `workspace:component-test`  | `cargo test -p winter-workspace-core --test '*'` | A behaviour against a fixture workspace, run in process with in-memory handles. | The behaviour reads or writes files or git state.          |
| `workspace:executable-test` | `cargo test -p winter-workspace --tests`         | Process-level contract: streams, exit code, environment.                        | The claim is about the process, not the library.           |
| `workspace:architecture`    | `cargo test -p winter-workspace-dev`             | Package edges, library output rules, context-tree rules, workflow and aliases.  | The change adds a package, an edge, or a context document. |
| `workspace:slice-gate`      | `cargo dev gate slice`                           | Formatting, lints, and every test but the recorded scenarios pass.              | Always, before handing a change over.                      |
| `workspace:full-gate`       | `cargo dev gate full`                            | The branch agrees with every recorded scenario.                                 | Before a merge and before a release.                       |
| `workspace:scenario`        | `cargo scenarios`                                | Equality with the recorded reference in exit code, JSON, and state.             | The behaviour exists in the reference tool.                |

No row says a change may be merged: a gate proves what its steps check, and the decision to merge is not the matrix's.

Manual testing.

```text
workspace:manual-gates
  setup and success: gates.md, Exercising the gates by hand — the document states both in full, since a contributor
  cannot open this spec
  use when: the change touches the gate driver, an alias, or the scenario target's manifest entry
workspace:manual-registration
  setup and success: command-registration.md, Exercising registration by hand — stated in full likewise
  use when: the change touches crates/core/build.rs, build/discover.rs, src/registry.rs, or src/parse.rs
workspace:manual-interrupt
  setup and success: SIGINT sent to the built binary while it waits in a write, once started with the default
  disposition and once with SIGINT ignored, with the endings of runtime.md, Interrupts and signals — stated in full
  likewise. It is what shows that `main` installs the handler: no test sends SIGINT to another process, by the
  signal rule context/standards/testing.md owns (System), to which the document links
  use when: the change touches crates/cli/src/interrupt.rs or the start of `main`
```

Every manual method runs in a scratch copy of the checkout under a temporary directory, never in the checkout itself,
and only where no directory at or above the scratch copy holds a `.winter/config.toml`; a method that starts the built
binary starts it by its path, as `context/tooling/development.md` says. Each entry's setup states where it may run, in
the words of the entry point's built-binary rule or by a link to it.

Tools.

| Tool             | Use                                                                                         |
| ---------------- | ------------------------------------------------------------------------------------------- |
| `tool:format`    | `cargo dev fmt` — rewrites formatting; run before the slice gate when its `fmt` step fails. |
| `tool:keep-tree` | `WINTER_TEST_KEEP=1` on any test command — keeps the isolated tree and prints its path.     |
| `tool:record`    | `cargo dev record` — added, with its use, by `epic:rust-adoption/parity-validation`.        |

```text
matrix rules
  ids         stable; a row is added, never renamed or reused
  real rows   a row exists only when its command exists and exits 0 on a clean checkout: `workspace:scenario` and
              `tool:record` are added by epic:rust-adoption/parity-validation with scenarios.md; before that the
              Commands part has six rows and the Tools part two
  one file    a test file narrower than a row is selected with `--test <subject>` in place of the row's test-target
              selection, `--test '*'` or `--tests`; the document says so once, under the Commands table
  replay      `workspace:executable-test` does not select the scenario replay target, which runs under the full gate
```

## Flows

```text
a contributor session
  AGENTS.md (always loaded) -> the standing rules hold
  task names a subject      -> routing table -> one area index -> one document
  change is ready           -> matrix.md -> the ids that verify it -> `cargo dev gate slice`
```
