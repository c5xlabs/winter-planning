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
    development.md                bootstrap, the gate commands, formatting, what the driver prints and exits with
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

## Interfaces

### Entry point

```text
AGENTS.md
  1. what the repository is, in one sentence
  2. standing rules — the rules a session must hold without opening another file:
       run `cargo dev gate slice` before handing a change over; run `cargo dev gate full` before a merge
       a command is one file under crates/core/src/commands/; no registry is edited
       the library returns values; only crates/cli/src/main.rs touches process streams, environment, and exit
       tests run in an isolated tree from winter-workspace-test-support; a test never touches a real workspace,
         the network, or a host-installed tool other than git
       nothing in the repository starts the reference tool; `cargo dev record` alone is handed its path
  3. routing table — one row per area index, each with "read when"
a rule is present only while the tree makes it true: the gate rule from the change that adds the gates, the command
rule from the change that adds registration, the test rule from the change that adds the harness, the last rule
from the change that adds `cargo dev record`
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
| `workspace:executable-test` | `cargo test -p winter-workspace --test '*'`      | Process-level contract: streams, exit code, environment.                        | The claim is about the process, not the library.           |
| `workspace:architecture`    | `cargo test -p winter-workspace-dev`             | Package edges, library output rules, context-tree rules, workflow and aliases.  | The change adds a package, an edge, or a context document. |
| `workspace:slice-gate`      | `cargo dev gate slice`                           | The change is mergeable.                                                        | Always, before handing a change over.                      |
| `workspace:full-gate`       | `cargo dev gate full`                            | The branch agrees with every recorded scenario.                                 | Before a merge and before a release.                       |
| `workspace:scenario`        | `cargo scenarios`                                | Equality with the recorded reference in exit code, JSON, and state.             | The behaviour exists in the reference tool.                |

Manual testing.

```text
workspace:manual-gates
  setup and success: gates.md, Exercising the gates by hand — the document states both in full, since a contributor
  cannot open this spec
  use when: the change touches the gate driver, an alias, or the scenario target's manifest entry
workspace:manual-registration
  setup and success: command-registration.md, Exercising registration by hand — stated in full likewise
  use when: the change touches crates/core/build.rs, build/discover.rs, src/registry.rs, or src/parse.rs
```

Both manual methods run in a scratch copy of the checkout under a temporary directory, never in the checkout itself;
each entry says so.

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
  one file    a test file narrower than a row is selected with `--test <subject>` in place of `--test '*'`; the
              document says so once, under the Commands table
```

## Flows

```text
a contributor session
  AGENTS.md (always loaded) -> the standing rules hold
  task names a subject      -> routing table -> one area index -> one document
  change is ready           -> matrix.md -> the ids that verify it -> `cargo dev gate slice`
```
