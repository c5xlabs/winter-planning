---
name: git-port
depends-on:
  - module-scaffold
---

# The git operations over the program port

Fills `git.rs` with the Git operations of [standalone-pins.md](../spec/standalone-pins.md), except `fetch`, with the
Operation spans there, over the program port of [child-processes.md](../../repository-catalog/spec/child-processes.md),
under `decision:git-cli`. Shaped by `engineering:domain-io-boundaries`: the operations take the ports a domain entry is
handed, and `Context` gains nothing. Nothing calls them yet.

## Change

- `head`, `clean`, and `verify` read; `checkout_branch`, `checkout_detached`, `stash_push`, and `stash_pop` write. Each
  is a function over `&dyn RunPrograms` and `&dyn CommandTracer` making `Program::Git` calls with the checkout as cwd; a
  domain entry passes them the `cx.programs` and `cx.tracer` its command file hands it.
- A non-zero exit is the `RepoError` of [location-listing.md](../../repository-catalog/spec/location-listing.md), except
  where the spec reads it as a value: `clean` reads any failure as dirty, and `verify` reads a non-zero exit as no
  match.
- Each operation opens its operation span through `CommandTracer::operation`, named and attributed as the spec gives it;
  `verify` opens none, since `resolve` opens the span its verifies share.
- The operations are re-exported by name from `lib.rs` for their component tests.

## Acceptance

- Component tests against the git fixtures, one per operation: `head` of a fresh clone, and of a directory that is not a
  repository, which fails; `clean` false for a modified tracked file, for an untracked file alone, and for a path that
  is not a repository; `verify` of a missing ref answers no match; `checkout_detached` of a commit leaves HEAD detached
  at it; `stash_pop` restores a stashed modification.
- A component test: a local commit on `main`, then `checkout_branch(main)`, leaves `main` at `origin/main` and the local
  commit off every branch.
- A component test: `stash_push` on a tree dirty only by an untracked file creates no stash entry.
- A component test with tracing on: one `git checkout` span per checkout, attributed with `winter.repo` alone.
- `cargo dev gate slice` exits 0.
