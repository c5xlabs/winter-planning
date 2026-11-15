---
name: runtime-documents
depends-on:
  - stdout-fault-tests
  - interrupt
code:
  - winter-workspace/context/architecture/structure.md
  - winter-workspace/context/verification/matrix.md
  - winter-workspace/README.md
---

# Contributor documents catch up with the runtime

Brings the context tree and the README in line with [contributor-context.md](../spec/contributor-context.md) and
[repository-layout.md](../spec/repository-layout.md) once the runtime is built, under
`engineering:workspace-development-context` and `product:local-component-evaluation` ("a contributor can build and run
the checkout's binary without replacing an installed command"). Each earlier change of the runtime adds its own module
row and rule; this one corrects what none of them owns. No source under `crates/*/src` changes. It follows
`stdout-fault-tests` and `interrupt`, which may be built beside each other, and is built before `command-span-record`;
it runs beside no other change.

## Change

- `context/architecture/structure.md`: the sentence that tells a contributor to start the binary with `cargo run` links
  to the built-binary rule of the entry point in its place; the package table gives each package the role
  [repository-layout.md](../spec/repository-layout.md), Packages and edges, gives it, the scenario machinery of the
  support package included; a dependency table is added, which the document does not hold yet, written from the four
  manifests as they stand, each dependency with its user. It matches the same spec's Cargo workspace table except for
  the rows `command-span-export` adds with its crates.
- `context/verification/matrix.md`: the `workspace:slice-gate` row's Proves cell is the one
  [contributor-context.md](../spec/contributor-context.md), Verifiability matrix, gives; each manual method's setup
  states the condition that spec gives for it, in the words of the entry point's built-binary rule or by a link to it.
  Every method id, each `**Setup.**` and `**Success.**` marker, and the phrase "scratch copy of the checkout" stay.
- `context/tooling/development.md`: Running the built binary keeps its link to the entry point's rule and restates none
  of it; the sentence that names bootstrap as the only step that needs the network stays true of every step the document
  lists.
- `README.md`: the status sentence of [repository-layout.md](../spec/repository-layout.md), Root after the wipe.
- `AGENTS.md` is not touched: its built-binary rule, its test rule, and its sentence on the driver's tests already say
  what [contributor-context.md](../spec/contributor-context.md), Entry point, says for the tree as it stands after
  `interrupt`.
- Files touched: `context/architecture/structure.md`, `context/verification/matrix.md`,
  `context/tooling/development.md`, `README.md`.
- `crates/dev/tests/architecture.rs` pins the matrix's method ids and manual-method markers, the documented commands,
  and the scope tokens, and `crates/dev/src/architecture.rs` holds the lists. The change keeps every method id and
  marker, adds no command to a document that the documented-commands list does not hold, and adds no scope token, so
  neither file changes.
- Scenarios: none.

## Acceptance

- `grep -rn 'cargo run' context/` prints nothing.
- `grep -rn 'mergeable' context/` prints nothing.
- `grep -c 'config.toml' context/verification/matrix.md` is at least 2, or each manual method links to
  `AGENTS.md#standing-rules`.
- Every module file under `crates/core/src` and `crates/cli/src`, other than command files, has a row in the Modules
  table of `context/architecture/structure.md`: the list `ls crates/core/src/*.rs crates/cli/src/*.rs` and the table's
  first column name the same files. Two exceptions: `crates/core/src/lib.rs` and `crates/cli/src/main.rs` have no row,
  since the document's prose describes each; and the rows for `commands/` and for `build.rs`, `build/discover.rs` name
  no file of the list.
- Every dependency line of `crates/{core,cli,test-support,dev}/Cargo.toml` has a row in the dependency table of
  `context/architecture/structure.md`, and the table has no other row.
- `git diff --stat` of the change names no file under `crates/` and not `AGENTS.md`.
- The architecture test's document checks pass: every context document is reachable from `AGENTS.md`, every relative
  link resolves, no scope token occurs, and every documented command is one the driver or Cargo has.
- `cargo dev gate full` exits 0.

## Out of scope

- A rule of the entry point itself: [contributor-context.md](../spec/contributor-context.md) owns its text.
- The loopback exception of the test rule: `command-span-record` adds it with the collector.
- Which built binary a contributor's shell selects: `decision:development-component-selection`, in
  `epic:platform/launcher-shim`.
- A manual of the binary's commands: `engineering:workspace-development-context` excludes it.
