---
name: process-harness
depends-on:
  - architecture-checks
---

# Isolated tree, pinned environment, and bounded process runner

Builds the tree, environment, and process sections of [test-harness.md](../spec/test-harness.md) in
`winter-workspace-test-support`, and the first executable test of the binary. Delivers the second and third acceptance
bullets of `engineering:testable-boundaries` and the isolation half of
`engineering:rust-adoption/parity-validation/disposable-runtime-proof`. Under
`decision:parallel-workspaces/component-runtime/harness-provenance` the code is written from the spec's interfaces
alone: the builder opens no source file of the platform repository. Under
`decision:parallel-workspaces/component-runtime/git-discovery-containment` git is kept inside the tree twice: by the
refusal at creation and by the pinned environment.

## Change

- `TempTree` creates the directory layout of the spec, refuses a parent with a workspace configuration at or above it,
  refuses a parent inside a git repository with a text of its own, refuses a root outside the path character set, and
  removes itself on drop unless `WINTER_TEST_KEEP=1`.
- `PinnedEnv` holds exactly the spec's variable table, `GIT_CEILING_DIRECTORIES` included, and computes its digest.
- `invoke` runs an absolute executable with a cleared environment, separate bounded capture of both streams, its own
  process group, and a deadline after which the group is killed and reaped. `InvocationFailure` and `FailureKind` have
  the spec's shape exactly: no private field, and the spawn cause on `FailureKind::Spawn`.
- The spec's "never passed" row is met by clearing the environment, not by a list of names: no file this change writes
  spells a name the spec's "names the package never spells" block forbids, in code, comment, or test.
- Every test of the package that fails by design follows the spec's "fixtures that fail by design" block: without
  `WINTER_TEST_FIXTURE` naming it, it does nothing.
- `crates/cli/tests/root.rs` launches the built binary through `invoke` and asserts the foundation parser answers of
  [command-registration.md](../spec/command-registration.md), Parser construction.
- `AGENTS.md` gains the standing rule about tests and the isolated tree.
- Files touched: `crates/test-support/Cargo.toml`, `crates/test-support/src/{lib,tree,env,process}.rs`,
  `crates/test-support/tests/{tree,process}.rs`, `crates/cli/Cargo.toml`, `crates/cli/tests/root.rs`, `AGENTS.md`,
  `Cargo.lock`.

## Acceptance

- A test creates a `TempTree`, asserts each directory and `gitconfig` exist, drops it, and asserts the root is gone;
  with `WINTER_TEST_KEEP=1` set on a child test process the root remains.
- With `WINTER_TEST_TMPDIR` naming a directory under a temporary `.winter/config.toml`, `TempTree::new` fails.
- With `WINTER_TEST_TMPDIR` naming a directory below one that holds `.git`, as a directory and then as a file,
  `TempTree::new` fails, creates nothing, and has `<the .git entry> is a git repository` in its error text. A directory
  inside a scratch bare repository is refused likewise, naming the bare repository. Neither text names a workspace
  configuration.
- A tree is created under a scratch directory inside another tree, and only then is the scratch directory made a git
  repository, so the refusal does not apply. Under the tree's pinned environment `git rev-parse --show-toplevel` exits
  non-zero in `<ROOT>/ws` and in `<ROOT>`, and `git commit --allow-empty -m probe` in `<ROOT>/ws` fails and leaves the
  scratch repository without a commit. After `git init` in `<ROOT>/ws`, `git rev-parse --show-toplevel` prints
  `<ROOT>/ws` there and in a directory below it.
- A test invokes `/usr/bin/env` and asserts the child's environment is exactly the pinned table: no variable of the test
  process leaks, including `WINTER_TEST_LEAK_PROBE`, which the test sets on itself first.
- A search for `WINTER_REFERENCE_BIN` over `crates/test-support`, `crates/core`, and `crates/cli`, and a
  case-insensitive search for `python` over `crates/test-support` and `crates/cli/tests/scenarios.rs`, find nothing.
- A test invokes `/bin/sh -c 'exit 7'` and reads `Exit::Code(7)` as an observation, not a failure.
- A test invokes a script that starts a background child and sleeps, with a 1 s timeout: the result is
  `FailureKind::Timeout`, and neither the script nor its child is alive afterwards.
- A test invokes a script that writes more than the output limit and reads `FailureKind::OutputLimit`.
- A relative executable is `FailureKind::InvalidInvocation` and starts nothing.
- An absolute executable that does not exist gives `FailureKind::Spawn` with empty `stdout` and `stderr`, holding the
  cause `<executable> in <working directory>: No such file or directory (os error 2)`. The test builds the expected
  failure as a literal and compares it for equality. The failure's text is `the executable could not be started:`
  followed by a space and the cause.
- Each fixture that fails by design, started in a child test process with `--ignored --exact` and without
  `WINTER_TEST_FIXTURE`, passes and leaves the directory `WINTER_TEST_TMPDIR` names empty.
- Two digests of two trees are equal; setting one variable changes the digest.
- `crates/cli/tests/root.rs`: `--version` prints `winter-workspace 0.1.0` on stdout with exit 0; `--help` exits 0 with
  empty stderr; no arguments exits 2 with empty stdout and, on stderr, the text `--help` printed on stdout; `--nope`
  exits 2 with empty stdout.
- `cargo dev gate slice` exits 0.

## Out of scope

- Git helpers and the workspace builder: `git-fixtures`.
- Scenario files, snapshots, and replay: `epic:rust-adoption/parity-validation`.
- The mechanical check of the forbidden names: `epic:rust-adoption/parity-validation`.
