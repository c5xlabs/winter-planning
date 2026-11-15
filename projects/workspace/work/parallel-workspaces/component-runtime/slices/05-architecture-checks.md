---
name: architecture-checks
depends-on:
  - gates
---

# Automatic checks of package edges, library rules, and gate wiring

Adds the architecture test of [repository-layout.md](../spec/repository-layout.md) — Packages and edges, Library rules —
and of [gates.md](../spec/gates.md), CI workflow. Delivers the edge-rejection clause of
`engineering:testable-boundaries` and the first acceptance bullet of `engineering:library-output-boundary`.

## Change

- `crates/dev/src/architecture.rs`, a module of the driver's library, holds each rule as a function over inputs the
  caller passes in, so a rule is itself tested against a violating input.
- `crates/dev/tests/architecture.rs` applies the rules to the checkout: it runs
  `cargo metadata --no-deps --locked
  --offline --format-version 1` through the `CARGO` path, parses it with
  `cargo_metadata`, and fails on a member set or a workspace edge the layout spec's table does not list, on a build edge
  between workspace packages, and on a missing `publish = false`.
- The same test scans `crates/core/src/**` and fails, naming file and line, on a forbidden token or a `mod.rs`;
  `std::fs` is forbidden everywhere but `crates/core/src/files.rs`, which need not exist yet.
- The same test fails when the `run:` lines or the `uses:` line of `.github/workflows/check.yml`, or the alias set of
  `.cargo/config.toml`, differ from the gates spec. The two files are read as that spec's "how the two files are read"
  says: the workflow by lines, the alias table with the `toml` crate.
- `crates/dev` gains `cargo_metadata` and `toml`, as the layout spec's dependency table gives them.
- Files touched: `crates/dev/tests/architecture.rs`, `crates/dev/src/{lib,architecture}.rs`, `crates/dev/Cargo.toml`,
  `Cargo.lock`.

## Acceptance

- `cargo test --locked -p winter-workspace-dev` passes on the checkout, and `cargo dev gate slice` exits 0.
- For each rule, a unit test feeds a violating input and asserts the failure names the offender: a fifth member; an edge
  from `winter-workspace-core` to `winter-workspace-test-support` of kind normal; a build edge; `println!` in a library
  source; `std::env::var` in a library source; `std::fs` in a library source other than `files.rs`; a `mod.rs`; a
  workflow with a fourth `run:` line; a workflow whose `uses:` line names a tag in place of the commit id; a workflow
  with a second `uses:` line; a third alias.
- A unit test passes `std::fs` inside `files.rs` to the token rule and gets no failure.
- The token scan reports a forbidden token inside a `#[cfg(test)]` module of the library.

## Out of scope

- The context-tree rules of [contributor-context.md](../spec/contributor-context.md): `contributor-context`.
- The error-type and `unsafe` rules of the layout spec, which review enforces.
