---
name: root-options
depends-on:
  - usage-diagnostic
code:
  - winter-workspace/crates/core/src/parse.rs::root_values
  - winter-workspace/crates/core/src/context.rs::Context
  - winter-workspace/crates/core/src/process.rs::run
---

# Root options, their placement rules, and the logging switch

Builds the root options and the Parsing rules of [command-surface.md](../spec/command-surface.md), Identity and root and
Parsing rules, and `RootArgs`, `RootOptions`, `root_values`, the orchestrator resolution, and `Log::resolve` of
[runtime.md](../spec/runtime.md), The parse and the run, What a command receives, and Logging, under
`product:component-help-version`, `product:discover-command-scope` ("help lists only commands the binary implements"),
and `engineering:handoff-fidelity`. It follows `usage-diagnostic` and is built before `diagnostics-and-logging`; it runs
beside no other change.

## Change

- `parse.rs`: the root holds `-v`/`--verbose`, `--service-orchestrator`, and the hidden `--source-override`, accepted
  only before the command path. Every value-taking option of the tree accepts a value that begins with `-`; a positional
  does not, and needs `--` before such a value. A flag or single-value option given twice ends as
  `ParseFault::RepeatedOption`, under `decision:parallel-workspaces/component-runtime/repeated-options`. A root given
  options and no command ends as the bare root does, under
  `decision:parallel-workspaces/component-runtime/help-rendering`. `Stage::Command` gains `root`, a `RootArgs` filled
  whether or not the command's own tokens parsed; `root_values` exists as the spec gives it.
- An argument that is not UTF-8 is refused before any argument is interpreted, as `ParseFault::NotUtf8`, under
  `decision:parallel-workspaces/component-runtime/non-utf8-arguments`, with the node at fault and the stage the spec
  gives.
- Help or version beside a fault ends as the Parsing rules give it, under
  `decision:parallel-workspaces/component-runtime/help-beside-fault`: the argument parser's own left-to-right order is
  kept, and nothing reorders it.
- `termination.rs`: `ParseFault::RepeatedOption` and `ParseFault::NotUtf8` exist and render.
- `crates/core/src/logging.rs` exists with `Level`, `Log`, `Log::DEFAULT`, and `Log::resolve`.
- `context.rs`, `process.rs`: `RootOptions` exists, with the attribute the command contract carries; `Context` gains
  `root` and `log`; step 2 of `run_with` takes both from `root_values`, and writes nothing yet.
- The scaffold gains `InProcess::os_args`.
- Contributor documents, each a sentence this change makes false or a rule it adds:
  - `context/architecture/structure.md`, Modules: the `logging.rs` row is added, and the `context.rs` row names
    `RootOptions`.
  - `context/architecture/changes.md`, A command: a value-taking option is declared so that it accepts a hyphen-leading
    value, with the name of the test that checks it.
  - `context/standards/rust.md`: the sentence on the items that carry `#[allow(dead_code…)]` permanently names five,
    `RootOptions` among them.
- Files touched: `crates/core/src/{parse,termination,context,process,lib}.rs`, `crates/core/src/logging.rs` (new),
  `crates/core/tests/support/in_process.rs`, `crates/core/tests/root.rs`, `crates/core/tests/{ws_index,repo_remove}.rs`,
  `crates/cli/tests/root.rs`, `context/architecture/{structure,changes}.md`, `context/standards/rust.md`, and the
  scenario directories below. No manifest changes.
- `crates/dev/tests/architecture.rs` pins nothing this change alters; `logging.rs` reads the environment only through
  `ProcessEnv`, so the forbidden-token scan stays green.
- Scenarios recorded with `cargo dev record`:
  - exit code alone: `root/version`, `root/help`, `root/bare`, `root/verbose-without-command`, `ws/bare`, `ws/help`,
    `ws-index/help`, `root/root-option-after-command`, `repo-remove/flag-value-refused`, and `root/fault-before-version`
    (`--bogus --version`);
  - with `stdout = "json"`: `ws-index/double-dash-name`, `ws-index/json-before-name`,
    `ws-index/with-orchestrator-override`;
  - deviation scenarios, each made by the flow of
    [record-and-replay.md](../../../../work/rust-adoption/parity-validation/spec/record-and-replay.md), "register a
    deviation": the scenario is written and recorded once, the `exit-code` file of `record/steps/01/` is copied to
    `expected/steps/01/` and edited to the decided value, and `[deviation]` names the decision:
    - `ws-index/repeated-flag`, whose one step is `-v -v ws index NAME` with text stdout: expected exit code 2,
      `decision:parallel-workspaces/component-runtime/repeated-options`;
    - `root/double-dash-before-command`, whose one step is `-- ws index NAME` with text stdout: expected exit code 2
      against a recorded 0, `decision:parallel-workspaces/component-runtime/double-dash-before-command`;
    - `root/version-beside-fault` (`--version --bogus`), `root/help-beside-fault` (`--help --bogus`), and
      `ws-index/help-beside-fault` (`ws index --help --bogus`), expected exit code 0,
      `decision:parallel-workspaces/component-runtime/help-beside-fault`. The reference tool accepts each of the five
      command lines and ends with another exit code, which is the one dimension these scenarios compare; that is what a
      deviation scenario holds. No step of the five asks for JSON, so no stdout document enters the comparison. What
      stdout carries in the three help and version cases is human text no scenario compares, so the component tests
      below hold it.
  - No scenario holds the refusal of an argument that is not UTF-8: a step's `args` are TOML strings and cannot spell
    one. The component tests and the executable test below hold it.

## Acceptance

- Unit tests of `logging.rs`: one case per row of the spec's `resolve` table, including each accepted level name in
  lower and upper case, `NOTSET` in both cases giving `Log { threshold: Level::Warning, switched: true }`, an unknown
  name, an empty value, a value that is not UTF-8, and `-v` beside each.
- A unit test over the registry: every value-taking option of every registered command, and of the root, accepts a
  hyphen-leading value; it fails naming the command and the option. No positional is given that setting:
  `ws index --bogus` stays an unknown option.
- Unit tests of `parse.rs`, each parsing a command line with the parser the registry builds and passing the `RootArgs`
  of the answer and a literal environment to `root_values`:
  - `-v ws index NAME` and `--verbose ws index NAME` return `Log { threshold: Level::Debug, switched: true }`, also with
    `WINTER_LOG_LEVEL=ERROR` set; `ws index NAME` returns `Log::DEFAULT`, and with `WINTER_LOG_LEVEL=info` returns
    `Log { threshold: Level::Info, switched: true }`;
  - `--service-orchestrator X ws index NAME` returns `service_orchestrator` of `Some("X")`; `--service-orchestrator ''`
    with `WINTER_SERVICE_ORCHESTRATOR=Y` returns `Some("Y")`; no flag and an empty variable returns `None`; a variable
    whose value is not UTF-8 returns those bytes;
  - `--service-orchestrator -x ws index NAME` returns `Some("-x")`;
  - `--source-override T ws index NAME` returns `source_override` of `Some("T")`, and `None` without the option;
  - `-v ws index --bogus` answers `Stage::Command` whose `RootArgs` has `verbose` set, though the parse failed;
  - `-v -v ws index NAME` answers `Stage::Root`.
- Component tests, through the scaffold:
  - `ws index -v NAME` and `ws index NAME --service-orchestrator X` end as a usage error naming `ws index`;
  - `-v`, `--service-orchestrator X`, and `-v --service-orchestrator X` with no command end as `BareGroup` with exit
    code 2 and the root help;
  - `-v -v ws index NAME`, `ws index NAME --json --json`, and
    `--service-orchestrator a --service-orchestrator b ws
    index NAME` end as `RepeatedOption`, exit code 2;
  - `--source-override` is absent from the root help text and accepted on the command line;
  - `--version --bogus` ends as `Success` with the version line on stdout and nothing on stderr; `--help --bogus` and
    `ws index --help --bogus` end as `Success` with that node's help on stdout; `--bogus --version`, `--bogus --help`,
    and `ws index --bogus --help` end as a usage error, exit code 2, with stdout empty; `ws index --help` with no `NAME`
    ends as `Success`;
  - `-- ws index NAME`, with `--` at the root, and `ws -- index NAME`, with `--` at a group, each end as a usage error,
    exit code 2, with stdout empty, under `decision:parallel-workspaces/component-runtime/double-dash-before-command`;
    `ws index -- NAME`, with `--` after the leaf, is accepted and looks up the name `NAME`;
  - through `InProcess::os_args`, an argument that is not UTF-8 as the name of `ws index`, as the value of
    `--service-orchestrator`, as the first argument, after `--` in `ws index -- <it>`, and after `--help` and after
    `--version` each end as `Termination::Usage` with `ParseFault::NotUtf8`, exit code 2, stdout empty, and no panic;
    the usage names `ws index` in the first and fourth case and the root in the others;
  - the root help lists exactly the groups and top-level commands the registry holds, and `init` ends as an unknown
    command, exit code 2.
- `crates/cli/tests/root.rs`: the built binary given an argument that is not UTF-8 exits 2 with empty stdout; and
  `--help` writes the same bytes with `COLUMNS=40` and with `COLUMNS=200` added to the pinned environment, with no line
  longer than 100 columns.
- `workspace:manual-registration` passes, since the change touches `crates/core/src/parse.rs`
  (`context/verification/matrix.md`).
- The eighteen scenarios pass, and `cargo dev gate full` exits 0.

## Out of scope

- Writing any log record: `diagnostics-and-logging`.
- What a service command does with the orchestrator override, and the override notice:
  `epic:running-environments/service-orchestration`.
- What `--source-override` selects: the epic that owns the extension sources.
