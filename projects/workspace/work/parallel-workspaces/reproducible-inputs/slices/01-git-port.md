---
name: git-port
depends-on:
  - epic:parallel-workspaces/repository-catalog
code:
  - winter-workspace/crates/core/src/context.rs::Context
---

# The git ports over the program port

Adds the read and write git ports of the Git operations in [standalone-pins.md](../spec/standalone-pins.md), except
`fetch`, with the Operation spans there, as one module over the program port of
[child-processes.md](../../repository-catalog/spec/child-processes.md), under `decision:git-cli`. Shaped by
`engineering:domain-io-boundaries`, which keeps read authority apart from mutation. Nothing calls the ports yet.

## Change

- `Context` gains a read git port (`head`, `clean`, `verify`) and a write git port (`checkout_branch`,
  `checkout_detached`, `stash_push`, `stash_pop`), each a `Program::Git` call through `RunPrograms` with the checkout
  as its cwd.
- A non-zero exit is the `RepoError` of location-listing.md, except where the spec reads it as a value: `clean` reads
  any failure as dirty, and `verify` reads a non-zero exit as no match.
- Each operation opens its operation span through the tracer port, named and attributed as the spec gives it; `verify`
  opens none, since `resolve` opens the span its verifies share.
- The ports are the module `git` (`crates/core/src/git.rs`), which only a domain entry may call: `git::` joins
  `COMMAND_FILE_NAMES` in `crates/dev/src/architecture.rs` and the matching row of the table in `changes.md`, and the
  Modules table of `structure.md` names it.
- The program port's types become `pub`, as the spec's visibility paragraph gives, and are re-exported with the ports.
- `standards/testing.md` gains the entry test: a component test that calls an entry re-exported from `lib.rs` against
  the git fixtures, for an entry that no command of the library reaches yet or that another epic's command calls.
- The ports are re-exported by name from `lib.rs` for their component tests.

## Acceptance

- Component tests against the git fixtures, one per operation: `head` of a fresh clone, and of a directory that is not
  a repository, which fails; `clean` false for a modified tracked file, for an untracked file alone, and for a path that
  is not a repository; `verify` of a missing ref answers no match; `checkout_detached` of a commit leaves HEAD detached
  at it; `stash_pop` restores a stashed modification.
- A component test: a local commit on `main`, then `checkout_branch(main)`, leaves `main` at `origin/main` and the local
  commit off every branch.
- A component test: `stash_push` on a tree dirty only by an untracked file creates no stash entry.
- A component test with tracing on: one `git checkout` span per checkout, attributed with `winter.repo` alone.
- A unit test of the command-file scan in `crates/dev`: a command file that names `git::` is a violation naming the
  file, the line, and the token.
- `cargo dev gate slice` exits 0.
