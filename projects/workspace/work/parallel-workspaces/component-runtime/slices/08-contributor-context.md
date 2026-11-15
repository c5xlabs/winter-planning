---
name: contributor-context
depends-on:
  - git-fixtures
---

# Context tree, routing, and the verifiability matrix

Writes the documents of [contributor-context.md](../spec/contributor-context.md) and the checks that keep them reachable
and in scope, delivering `engineering:workspace-development-context`. Every statement in the tree restates a rule of
[repository-layout.md](../spec/repository-layout.md), [command-registration.md](../spec/command-registration.md),
[gates.md](../spec/gates.md), or [test-harness.md](../spec/test-harness.md) for a contributor, and names no command of
the product beyond running the binary from the checkout.

## Change

- `AGENTS.md` keeps the four standing rules earlier changes added and gains the routing table; `context/index.md`, the
  four area indexes, and the documents `architecture/structure.md`, `architecture/changes.md`, `standards/rust.md`,
  `standards/testing.md`, `tooling/development.md`, and `verification/matrix.md` exist. The routing table and the
  verification index have no row for `scenarios.md`.
- `verification/matrix.md` has the three parts of the spec's Verifiability matrix: six Commands rows; the two manual
  entries, `workspace:manual-gates` with its setup and success condition written out from [gates.md](../spec/gates.md),
  Exercising the gates by hand, and `workspace:manual-registration` with its own written out from
  [command-registration.md](../spec/command-registration.md), Exercising registration by hand; and the Tools rows
  `tool:format` and `tool:keep-tree`.
- The architecture test gains the context-tree rules: reachability from `AGENTS.md`, resolving links, and the scope
  check over the files and tokens the spec lists.
- Files touched: `AGENTS.md`, `context/**`, `crates/dev/src/architecture.rs`, `crates/dev/tests/architecture.rs`.

## Acceptance

- The architecture test passes, and for each context rule a unit test feeds a violating tree and asserts the failure
  names the file: a document no index links, a link to a missing file, a document containing `PyTest`, an `AGENTS.md`
  containing `uv run`; a `context/verification/scenarios.md` containing `python` is not reported.
- Every command line in a fenced block of `tooling/development.md` and every command of the matrix's Commands and Tools
  parts is one of the commands of the gates spec or a `cargo test -p <package>` call naming an existing package; a test
  extracts them and checks them against that list.
- The Commands part has exactly the ids `workspace:unit-test`, `workspace:component-test`, `workspace:executable-test`,
  `workspace:architecture`, `workspace:slice-gate`, `workspace:full-gate`; a test reads them from the file.
- The Manual testing part has exactly the ids `workspace:manual-gates` and `workspace:manual-registration`; each states
  its setup, its success condition, and that it runs in a scratch copy of the checkout; a test reads the two ids from
  the file.
- Each of the six Commands rows' command exits 0 on the clean checkout.
- `cargo dev gate full` exits 0.

## Out of scope

- `verification/scenarios.md`, the `workspace:scenario` row, and `tool:record`: `epic:rust-adoption/parity-validation`.
- The `context` command of the binary: `epic:agent-ready-operation/context-navigation`.
- Formatting and lint of the Markdown itself: excluded by `decision:parallel-workspaces/component-runtime/gate-scope`.
