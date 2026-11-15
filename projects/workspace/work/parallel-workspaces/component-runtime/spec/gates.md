# Gates

The two verification gates of the code repository, the driver that runs them, the Cargo aliases, and the CI workflow.
The package layout is [repository-layout.md](./repository-layout.md); the scenario target the full gate runs is defined
by the [record and replay spec](../../../rust-adoption/parity-validation/spec/record-and-replay.md).

## Interfaces

### Commands

```text
# engineering:verification-gates, decision:parallel-workspaces/component-runtime/gate-driver
# decision:parallel-workspaces/component-runtime/gate-scope — Cargo steps only; no Markdown or workflow lint
# run from the repository root
cargo dev gate slice      the slice gate
cargo dev gate full       the full gate
cargo dev fmt             rewrite formatting; the only driver command that changes a tracked file
cargo scenarios [FILTER]  the recorded scenarios alone; FILTER is a substring of a scenario id
cargo fetch --locked      dependency bootstrap, once per checkout and after Cargo.lock changes; needs the network
```

```toml
# .cargo/config.toml
[alias]
dev = "run --quiet --locked --offline -p winter-workspace-dev --"
scenarios = "test --locked --offline -p winter-workspace --features scenarios --test scenarios --"
```

### Steps

Every step is one child process whose program is the path in the `CARGO` environment variable, which Cargo sets for the
driver it runs. The driver starts no other program.

| Step        | Gate        | Arguments after the Cargo path                                                                          |
| ----------- | ----------- | ------------------------------------------------------------------------------------------------------- |
| `fmt`       | slice, full | `fmt --all --check -- <the explicit file list of command-registration.md, Formatting discovered files>` |
| `clippy`    | slice, full | `clippy --locked --offline --workspace --all-targets --all-features -- -D warnings`                     |
| `test`      | slice, full | `test --locked --offline --workspace`                                                                   |
| `scenarios` | full        | `test --locked --offline -p winter-workspace --features scenarios --test scenarios`                     |

`cargo dev fmt` runs `fmt --all -- <the same file list>`.

```text
order         as listed; a later step runs whether or not an earlier one failed
warnings      denied in the clippy step alone: `-- -D warnings` there covers compiler warnings and Clippy warnings of
              every workspace package, every target, and every feature, the scenario target included.
              The driver neither sets nor removes RUSTFLAGS or CARGO_ENCODED_RUSTFLAGS, so the test steps reuse the
              build the driver itself was compiled in and nothing is compiled twice for a flag difference.
environment   each step inherits the driver's environment unchanged
output        each child's stdout and stderr pass through unchanged as they are produced
summary       last line on stderr:  "gate <name>: ok"   or   "gate <name>: failed: <step>[, <step>…]"
exit          0 all steps passed · 1 any step failed, timed out, or could not start · 2 unknown driver arguments
deadline      15 minutes per step; on expiry the step's process group is killed and reaped and the step is failed
interrupt     SIGINT or SIGTERM kills the running step's process group and exits 1
writes        nothing outside target/ ; `gate` never passes --fix or omits --check
network       none: every Cargo call that resolves dependencies carries --locked --offline
```

```rust
// crates/dev/src/lib.rs — the driver is a library with a thin binary (repository-layout.md), so its plan is testable
pub struct Step { pub name: &'static str, pub program: PathBuf, pub args: Vec<OsString> }
pub enum Gate { Slice, Full }
pub const STEP_DEADLINE: Duration = Duration::from_secs(15 * 60);
pub fn plan(gate: Gate, cargo: &Path, repository_root: &Path) -> Vec<Step>;      // pure but for listing commands/
// Runs all steps, each bounded by `deadline`, and prints the summary. main_with passes STEP_DEADLINE; a test
// passes a short one.
pub fn run_steps(name: &str, steps: &[Step], deadline: Duration, err: &mut dyn Write) -> u8;
// The driver's one entry. `env` looks up one variable of the driver's environment: main.rs passes
// std::env::var_os, a test passes a closure, and no driver command reads the process environment another way.
// The gates read CARGO through it. Arguments that name no command the driver has are unknown: exit 2 — so a
// driver that has no command yet answers 2 to every argument vector.
pub fn main_with(args: &[OsString], env: &dyn Fn(&str) -> Option<OsString>, root: &Path) -> u8;
```

### Which check admits a change

A change is admitted by the strongest check its own tree can run.

| Tree the change produces                                      | Check                                                                                 |
| ------------------------------------------------------------- | ------------------------------------------------------------------------------------- |
| No `Cargo.toml` at the root                                   | The change's own acceptance checks                                                    |
| A Cargo workspace without the `dev` alias and the gate driver | `cargo build --workspace --locked` and `cargo test --workspace --locked`, both exit 0 |
| The `dev` alias and the gate driver are present               | `cargo dev gate slice` exits 0                                                        |

A change made before the gate driver exists still leaves a tree that the driver's `fmt` and `clippy` steps would pass:
its author runs `cargo fmt --all --check` and
`cargo clippy --locked --workspace --all-targets --all-features -- -D warnings` by hand. The change that adds the driver
therefore adds it to a tree that already passes, and repairs nothing.

### Exercising the gates by hand

```text
The gate's own failure behaviour cannot be a test of the suite it runs. It is exercised by hand, in a scratch copy of
the checkout under a temporary directory (never in the checkout itself), once per fault:
  a mis-indented line in a library source        cargo dev gate slice -> exit 1, summary names fmt
  an unused `let` binding in a library source    exit 1, summary names clippy          (compiler warning)
  `let _x = 1u8.clone();` in a library source    exit 1, summary names clippy          (Clippy warning)
  a new unit test in a library source whose body is `assert_eq!(std::process::id(), 0);`
                                                 exit 1, summary names test; fmt and clippy not named
  an empty directory scenarios/x-y/z/ added      cargo scenarios -> exit 1, naming the directory;
                                                 cargo dev gate full -> exit 1, names scenarios;
                                                 cargo dev gate slice -> exit 0
Success: each run ends with the stated exit and summary, every later step still ran, and `git status` of the scratch
copy shows only the injected edit (the last fault adds no tracked file, so there it shows nothing).
The fourth fault fails when the test runs and at no other step: a process id is never 0, and the condition is not a
constant, so the clippy step, which lints test code as well, reports nothing about it. An assertion on a constant —
`assert!(false)` — would fail the clippy step too and is not used.
The last fault needs no edit to the scenario target: the target fails on a scenario directory it cannot replay,
which an empty one is, both while the target is the stand-in and once it replays.
```

### The scenario target

```toml
# crates/cli/Cargo.toml
[features]
scenarios = []          # selects the test target below; no source file tests cfg(feature = "scenarios")

[[test]]
name = "scenarios"
path = "tests/scenarios.rs"
harness = false
required-features = ["scenarios"]
```

```text
`cargo test --workspace` does not build or run the target, because the feature is off; that is the whole difference
between the slice gate's test step and the full gate.
`cargo clippy … --all-features` lints it in both gates.
The target receives the binary's path from Cargo as CARGO_BIN_EXE_winter-workspace and the scenario root as
<repository root>/scenarios. With no scenario under that root it reports zero scenarios and exits 0; a scenario
it cannot replay fails it.
```

### CI workflow

```yaml
# .github/workflows/check.yml — decision:parallel-workspaces/component-runtime/ci-workflow
# engineering:reproducible-build: CI runs the command a contributor runs
name: check
on: [push, pull_request]
permissions:
  contents: read
jobs:
  gate:
    runs-on: ubuntu-24.04
    timeout-minutes: 60
    steps:
      - uses: actions/checkout@3d3c42e5aac5ba805825da76410c181273ba90b1 # v7.0.1
      - run: rustup toolchain install          # reads rust-toolchain.toml
      - run: cargo fetch --locked
      - run: cargo dev gate full
```

```text
checked by crates/dev/tests/architecture.rs:
  the workflow's `run:` lines are exactly the three above, in that order
  the workflow has exactly one `uses:` line, and it is the one above: the action at that full commit id
  .cargo/config.toml defines exactly the aliases `dev` and `scenarios`
how the two files are read
  the workflow    as lines, with no YAML parser: a `run:` or `uses:` line is one whose text, after leading spaces
                  and an optional "- ", starts with that key; its value ends before a " #" comment
  the aliases     the key set of the [alias] table of .cargo/config.toml, parsed with the `toml` crate
```
