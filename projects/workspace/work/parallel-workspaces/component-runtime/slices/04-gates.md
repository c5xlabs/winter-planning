---
name: gates
depends-on:
  - command-registration
---

# The slice gate, the full gate, and the workflow that runs it

Builds [gates.md](../spec/gates.md): the driver, the two aliases, the scenario test target, and the CI workflow.
Delivers `engineering:verification-gates` and the CI bullet of `engineering:reproducible-build` under
`decision:parallel-workspaces/component-runtime/gate-driver`,
`decision:parallel-workspaces/component-runtime/gate-scope`, and
`decision:parallel-workspaces/component-runtime/ci-workflow`.

## Change

- `cargo dev gate slice`, `cargo dev gate full`, and `cargo dev fmt` exist and behave as the spec's Steps section says:
  every step runs, the summary line names the failed ones, the exit code is 0, 1, or 2. The driver is the library target
  of `crates/dev` with the spec's `plan`, `run_steps`, `STEP_DEADLINE`, and `main_with`; `src/main.rs` stays thin.
- Every step's program is the `CARGO` path, read through the environment lookup `main_with` is given; a missing `CARGO`
  variable is exit 1 with a message naming it. The driver sets no `RUSTFLAGS`.
- The format step passes the explicit file list of [command-registration.md](../spec/command-registration.md),
  Formatting discovered files.
- `crates/cli` declares the `scenarios` feature and test target; `tests/scenarios.rs` is a stand-in: it prints
  `scenarios: 0 passed, 0 failed, 0 deferred` and exits 0 when `scenarios/` is absent or holds no directory, and exits 1
  naming the first directory when one exists.
- `.github/workflows/check.yml` exists as the spec's CI workflow block gives it, the checkout action at the commit id
  written there. It is not run by this change.
- `AGENTS.md` gains the standing rule about the two gates.
- Files touched: `.cargo/config.toml`, `.github/workflows/check.yml`, `AGENTS.md`, `crates/dev/Cargo.toml`,
  `crates/dev/src/{lib,main,gate,fmt,step}.rs`, `crates/cli/Cargo.toml`, `crates/cli/tests/scenarios.rs`, `Cargo.lock`.
- The tree this change starts from already passes the format and Clippy steps; a failure of either on files this change
  did not write is a fault to report, not to repair here. From this change on, every change is admitted by
  `cargo dev gate slice`.

## Acceptance

- `cargo dev gate slice` and `cargo dev gate full` exit 0 on the clean checkout and leave `git status` clean; each
  prints `gate <name>: ok` as its last stderr line.
- A unit test of `run_steps` with steps whose programs are `/bin/sh` scripts: with the second of three failing, the
  third still runs, the summary is `gate slice: failed: <second>`, and the result is 1; with a one-second `deadline`
  passed in and a step that sleeps longer, the step is failed and no process of its group is alive.
- A unit test asserts `plan` for each gate equals the spec's table: names, order, and argument vectors, every `program`
  equal to the Cargo path passed in, `--locked --offline` on the Clippy and test steps, and `--check` on the format
  step.
- `cargo dev gate nope` exits 2.
- A unit test of `main_with` with an environment lookup that has no `CARGO` reads 1 for `gate slice`.
- A unit test of the format file list returns `build/discover.rs` and every `.rs` file under a temporary `commands/`
  tree, sorted, and `build/discover.rs` alone when the directory is absent.
- `cargo test --locked --workspace` does not build the `scenarios` target; `cargo scenarios` builds and runs it.
- The `uses:` line of `.github/workflows/check.yml` is the spec's, character for character.
- The five runs of the spec's Exercising the gates by hand, made in a temporary copy of the checkout, each end with the
  stated exit and summary; the fifth is the one that shows the stand-in target exiting 1 on a `scenarios/x-y/z/`
  directory. They are this change's manual method: the gate's own failure paths have no automatic check.

## Out of scope

- The replay runner that replaces the stand-in target: `epic:rust-adoption/parity-validation`.
- `cargo dev record`: `epic:rust-adoption/parity-validation`.
- Markdown and workflow lint: excluded by `decision:parallel-workspaces/component-runtime/gate-scope`.
- The check that the workflow and aliases keep their specified content: `architecture-checks`.
