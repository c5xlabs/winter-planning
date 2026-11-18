# Record and replay

The two operations over a scenario directory: the record step, which runs the reference tool once and stores what it
did, and replay, which runs the binary against the same fixture and compares. The files both read and write are
[scenario-format.md](./scenario-format.md); the gate that runs replay is
[gates.md](../../../../delivered/parallel-workspaces/component-runtime/spec/gates.md).

## System

```text
# decision:rust-adoption/parity-validation/recorder-placement
crates/test-support/src/scenario.rs       facade of the scenario modules
crates/test-support/src/scenario/
  model.rs        scenario.toml, reference.toml, meta.toml as types; discovery of scenario directories
  fixture.rs      builds a TempTree from a scenario's fixture description
  snapshot.rs     tree.txt, git.txt, files/ from a tree
  normalize.rs    ROOT replacement, JSON stream parsing and canonical encoding
  compare.rs      record or expectation against an observation -> Vec<Difference>
  stub.rs         stub programs and their call records; present once call capture exists
  runner.rs       run one scenario's steps with one executable; replay of a scenario set
  record.rs       record one scenario with a given executable: two trees, stability check, write record/
crates/dev/src/record.rs                  `cargo dev record`: arguments, refusals, the reference-only environment;
                                          the only code that is given the reference tool's path
crates/cli/tests/scenarios.rs             the replay test target; calls runner::replay_all and exits with its code
```

```text
who runs what
  record   crates/dev  -> scenario::record(executable = the reference) -> fixture, runner, snapshot, normalize -> record/
  replay   crates/cli test target -> fixture -> runner(executable = the binary) -> snapshot -> normalize -> compare
the reference tool is an opaque executable path; no code reads its sources, imports it, or knows its language
tests of the scenario modules record and replay with stand-in executables they write themselves
```

```text
# engineering:record-step-separation — checked by crates/dev/tests/architecture.rs, as substring rules over file text
WINTER_REFERENCE_BIN                    occurs in no file under crates/cli, crates/core, crates/test-support
python (ASCII case-insensitive)         occurs in no file under crates/test-support, nor in crates/cli/tests/scenarios.rs
replay                                  runner::replay_all passes no extra_env: the binary's environment is the pinned
                                        table of the test harness and the step's own `env`, nothing else
what the two rules protect              no gate and no test can start the reference tool: the variable that carries
                                        its path is unknown outside crates/dev, and the code that builds fixtures,
                                        runs steps, records, and replays cannot name its language
outside the word rule                   crates/core and crates/cli/src. The product names that language where a
                                        contract of its own does — an interpreter variable set or dropped in a
                                        child's environment, an extension entry point — and the rule protects
                                        nothing there.
```

The harness side of the two substring rules is stated where the harness is specified:
[test-harness.md](../../../../delivered/parallel-workspaces/component-runtime/spec/test-harness.md), The pinned
environment, "names the package never spells". The trees the rules cover therefore already satisfy both when the check
arrives, and the change that adds the check repairs nothing.

## Interfaces

### Record

```text
# engineering:record-step-separation
cargo dev record <scenario id>… [--reference <path>] [--rerecord] [--keep]
cargo dev record --all-missing   [--reference <path>]

--reference <path>    the reference executable; else the WINTER_REFERENCE_BIN variable; neither -> exit 2, nothing built
--rerecord            replace an existing record/ of the named scenarios; never combined with --all-missing
--keep                leave both fixture trees on disk and print their paths
exit                  0 every named scenario recorded and the reference checkout is unchanged
                      1 any scenario refused or unstable, or the reference checkout changed during the run
                      2 arguments
environment           the command reads WINTER_REFERENCE_BIN and PATH through the lookup the driver's main_with is
                      given (gates.md, Steps), never from the process directly, so a test passes both
```

```text
refusals, each before the reference is started, each naming what it found
  the reference path is relative, missing, or not executable
  the reference executable is the first `winter` on the caller's PATH        # the installed command is never run
  the reference's checkout cannot be found by walking up from the executable to a .git entry
  that checkout's canonical path does not match [A-Za-z0-9/._-]+            # it is replaced textually; Normalization
  that checkout's HEAD is not scenarios/reference.toml `commit`, or its tracked files are modified
  the temporary root has .winter/config.toml at or above it                 # TempTree refuses; test-harness.md
  the scenario has record/ and --rerecord is absent
  the scenario is malformed — a path that is not confined included (scenario-format.md, Path confinement) —
    or has a [[stub]] while runner::CALL_CAPTURE is false
order
  every named scenario is parsed, and a malformed one refused, before the reference is first started — its
  --version call included — and before any tree is created. A run in which every named scenario is refused
  never starts the reference and creates no tree.
a scenario with a non-empty `needs` the runner cannot satisfy is skipped with a line saying so; exit is unaffected
"the same executable" is decided by comparing canonical paths (symlinks resolved) of the two files
```

```text
the reference checkout is read, never written
  the recorder runs exactly three git commands on it, and nothing else touches it:
    git -C <checkout> rev-parse HEAD
    git -C <checkout> status --porcelain=v2 --untracked-files=no
    git -C <checkout> ls-files --others -z
  each is run twice: once before the reference is first started — its --version call included — and once after the
  last scenario of the run has been recorded, refused, or skipped. A run that ends at a refusal before the reference
  was ever started makes only the first pass.
  all with a cleared environment holding only
    PATH=<directory of git>:/usr/bin:/bin   HOME=<a temporary directory>   GIT_CONFIG_GLOBAL=/dev/null
    GIT_CONFIG_NOSYSTEM=1   GIT_OPTIONAL_LOCKS=0   LC_ALL=C.UTF-8
  GIT_OPTIONAL_LOCKS=0 keeps `git status` from refreshing and rewriting the index: the bytes and the modification
  time of <checkout>/.git/index, or of the index of the worktree the .git file points at, are the same after a
  record as before it. `ls-files --others` reads the working tree and writes nothing.
  the reference itself runs with cwd inside the fixture tree and the bytecode switch below, and is given no path
  into its own checkout. The cwd rule is scenario-format.md, Path confinement, held a second time by the runner
  immediately before each spawn (Runner, "confinement"): the reference finds its workspace by walking up from its
  cwd, so a cwd outside the tree would be a real workspace.
```

```text
the closing check: the second pass against the first
  ls-files --others, without an exclude option, lists every file git does not track, the git-ignored ones included,
  one path per file. A reference checkout normally holds such files already — an installed environment, caches —
  so the check never expects the list to be empty: it compares the two lists as sets.
  unchanged     the same HEAD; the status output still empty; the same set of untracked paths
  changed       a different HEAD, a tracked file now modified, or a path added to or removed from the set
                -> one line "reference checkout changed" on stderr, then each added path with "+", each removed
                   path with "-", each modified tracked file with "M", at most twenty lines and a count of the rest;
                   exit 1. The records the run wrote stay on disk and are not to be kept: the reference that made
                   them was not running the way this spec requires.
  not seen      a change to the content of a file that was untracked before the run. The bytecode switch and the
                cwd rule above are what prevent that; this check catches a new file, a removed one, and any
                tracked change.
```

```rust
// crates/test-support/src/scenario/record.rs
pub struct RecordOptions {
    pub executable: PathBuf,
    pub extra_env: Vec<(String, String)>,     // reference-only additions; not part of the digest
    pub redactions: Vec<(PathBuf, String)>,   // each path, as bytes, becomes the token in everything stored, after ROOT
    pub reference_commit: String,
    pub reference_runtime: String,
    pub rerecord: bool,
    pub keep: bool,
}
pub enum RecordError {
    Malformed(/* … */), Exists, Fixture(/* … */), Run(/* … */),
    Unstable(Difference),        // the first difference between the two roots, under the comparison rules
}
pub fn record(scenario_dir: &Path, options: &RecordOptions) -> Result<(), RecordError>;
```

```text
record one scenario
  1. build the fixture in tree A; snapshot (before of step 01)
  2. for each step: invoke the executable under the pinned environment with the step's args, cwd, stdin, env;
     snapshot after
  3. repeat 1–2 in a second tree B with a different root
  4. normalize both; compare B against A with compare::differences — the function and the scenario's [compare]
     rules replay uses, A standing as the record. Any Difference -> RecordError::Unstable with the first one;
     nothing is written. Text stdout and stderr are not compared, exactly as at replay: a record is unstable only
     in a dimension that could fail a gate.
  5. write record/ from A; write meta.toml
environment additions `cargo dev record` passes as extra_env
  the two interpreter variables that switch off bytecode files and fix the hash seed, each set to its documented
  value (1 and 0); crates/dev/src/record.rs names them
  PATH gains the directory of the reference executable, after <directory of git>
redactions `cargo dev record` passes
  one: the canonical path of the reference checkout, with the token <REFERENCE> (scenario-format.md, Normalization)
reference_runtime
  the first stdout line of `<reference> --version`, run once through `invoke` under a TempTree's pinned
  environment with those additions and cwd <ROOT>/ws; a failure to run it is a refusal
```

### Replay

```text
# engineering:behavioral-fixtures, engineering:parallel-workspaces/component-runtime/verification-gates
cargo scenarios [FILTER]…          every scenario whose id contains any FILTER; all when none is given
  --list                           print ids with their state (recorded, deferred, unrecorded) and exit 0
  WINTER_TEST_KEEP=1               keep the tree of every failed scenario and print its path
  WINTER_SCENARIO_JOBS=<n>         worker count; default min(available parallelism, 8)
exit                               0 no scenario failed · 1 otherwise
```

```text
output, on stdout
  one line per scenario:   "ok <id>"  ·  "deferred <id> needs <facility>[,<facility>…]"  ·  "FAILED <id>"
  under a FAILED line:     one block per Difference — dimension, step, subject, then recorded and observed values,
                           a unified diff for text, a pointer and both values for JSON
  last line:               "scenarios: <p> passed, <f> failed, <d> deferred"
what fails a run
  any Difference in any scenario
  a malformed scenario; a scenario directory without record/ ("unrecorded")
  a record whose reference_commit, environment, or description digest does not match (stale-record)
  the fixture could not be built; the binary timed out or exceeded the output limit
what never fails a run
  a deferred scenario; a difference in stdout or stderr text
```

### Comparison

```rust
// crates/test-support/src/scenario/compare.rs
pub enum Dimension { Exit, Json, Tree, Files, Git, Calls, Fixture, StaleRecord, Malformed }
pub struct Difference {
    pub dimension: Dimension,
    pub step: Option<u32>,
    pub subject: String,        // a JSON pointer, a ROOT-relative path, a repository and block, a call number
    pub recorded: String,
    pub observed: String,
}
// `recorded` and `observed` are both the in-memory form of record/steps/** ; replay loads the first from disk,
// the record step passes tree A.
pub fn differences(scenario: &Scenario, recorded: &[StepRecord], observed: &[StepRecord]) -> Vec<Difference>;
```

| Dimension     | Compared                                                                                | Equal when                                                                                                             |
| ------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `Exit`        | `exit-code`                                                                             | the same code, or the same signal.                                                                                     |
| `Json`        | `stdout.json`, steps whose stdout mode is `json` or `json-lines`                        | the same number of values, pairwise equal as JSON values, in order, after the rules below.                             |
| `Tree`        | `after/tree.txt`                                                                        | the same entries; content hashes are compared only for paths whose file mode is `bytes`.                               |
| `Files`       | `after/files/**`                                                                        | per path, by its mode: `bytes` equal bytes · `toml` equal parsed TOML values · `presence` both exist · `ignore` never. |
| `Git`         | `after/git.txt`                                                                         | block for block equal text.                                                                                            |
| `Calls`       | `calls/*.json`                                                                          | the same number of calls, pairwise equal in program, argv, cwd, env.                                                   |
| `Fixture`     | `before/` of step 01 against the snapshot of the freshly built fixture                  | equal; a difference means the builder or git changed, not the binary.                                                  |
| `StaleRecord` | `meta.toml` against `reference.toml`, `PinnedEnv::digest()`, and the description digest | all three equal.                                                                                                       |
| `Malformed`   | the scenario directory against scenario-format.md                                       | it parses, every path it names is confined, and it has no `[[stub]]` while call capture is absent.                     |

```text
JSON rules
  reading         stdout.json is the canonical form of the stream, and that form merges streams that differ. Each
                  side is therefore read on its own. A side whose stdout.json is the canonical form of its own
                  stdout is read again from that stdout, with every number kept as it is written and every member
                  of an object kept: integers beyond 64 bits, a repeated key, and a line {"$text": …} against a
                  line that is not JSON all differ. A side whose stdout.json is not that form — a record whose
                  stdout.json an expectation replaced — is read from its stdout.json as that is written, with its
                  numbers and members kept likewise. What the canonical form merges in an observation therefore
                  still differs from an expectation, and an expectation states a number as the observation
                  prints it, not as the canonical form rounds it.
  message text    a string at a pointer matched by [compare] message_text is equal to any string;
                  under json-lines a line that is not JSON is equal to any other such line. Where a side is read
                  from its stdout.json, an object whose one member is a string at "$text" stands for such a
                  line; under json no value stands for a line.
  fan-out         within a maximal run of consecutive values whose "type" is in a [[compare.fan_out]] `types`,
                  values are partitioned by the value at `key`; the partitions are compared as a set, and the
                  values inside one partition in order
  numbers         compare by their value as decimals of any length: 1, 1.0, and 1e0 are equal
text never compared
  stdout and stderr of every step are stored and shown beside a failure; when stdout mode is "text" the step's
  stdout contributes no Difference
before the binary's first step
  the Fixture dimension is checked first; when it differs the scenario fails with that one Difference and the
  binary is not run
```

```text
comparison rules — the values of [compare] in scenario.toml, read by compare::differences before a step is compared
  malformed         a rule that does not read -> one Difference of Dimension::Malformed for each such rule, without
                    a step, whose subject is the rule's key; nothing else is compared
                      message_text        a pointer that is neither empty nor begins with "/", or that holds a "~"
                                          followed by neither "0" nor "1"
                      key                 of a [[compare.fan_out]]: a pointer with either fault
                      types               of a [[compare.fan_out]]: an empty list, or a type listed twice, in the
                                          same fan-out or in another
                      path                of a [[compare.file]]: no confined relative path (scenario-format.md,
                                          Path confinement), or no glob — an empty segment, a segment "." or
                                          "..", or "**" beside something else in a segment
                      git_config_ignore   no key git could have written: it holds no ".", or holds an "=" or a
                                          line break
  relaxes nothing   a rule that reads and relaxes nothing in the scenario -> one Difference without a step, in the
                    dimension the rule would have relaxed, beside every other Difference of the comparison
                      message_text        no string of the recorded output is at the pointer
                                          Json, subject "message_text <pointer>"
                      [[compare.fan_out]] no value of the recorded output has one of its types
                                          Json, subject "fan_out <types>"
                      [[compare.file]]    it decides the mode of no path either side lists after a step; a rule
                                          an earlier one always matches first decides none
                                          Files, subject "file <glob>"
                      git_config_ignore   not reported: its keys are left out when a snapshot is taken, so two
                                          snapshots cannot tell whether a key names anything
```

### Runner

```rust
// crates/test-support/src/scenario/runner.rs
pub const SUPPORTED: &[Facility] = &[];
pub const CALL_CAPTURE: bool = false;        // true once stub.rs exists
pub struct ReplayOptions { pub executable: PathBuf, pub scenarios_root: PathBuf, pub filters: Vec<String>, pub jobs: usize }
pub struct ReplayReport { pub passed: Vec<String>, pub failed: Vec<(String, Vec<Difference>)>, pub deferred: Vec<(String, Vec<Facility>)> }
pub fn replay_all(options: &ReplayOptions, out: &mut dyn std::io::Write) -> ReplayReport;
pub fn run_steps(scenario: &Scenario, tree: &TempTree, executable: &Path, extra_env: &[(String, String)])
    -> Result<Vec<StepRecord>, RunFailure>;
pub enum RunFailure {
    Unconfined { step: u32, cwd: PathBuf },  // the step's cwd does not resolve inside the tree; nothing was started
    /* … one variant per FailureKind of the harness's invoke */
}
```

```text
step deadline     30 s; a timeout is a failed scenario at replay and a refused record at record time
isolation         one TempTree per scenario; scenarios share nothing and run on worker threads
determinism       scenario ids are sorted; output lines are printed in id order whatever order workers finish in
confinement       run_steps trusts no caller. Immediately before each spawn it canonicalizes the working directory
                  it is about to pass to `invoke` and requires an existing directory under the canonical
                  tree.root(); otherwise it returns RunFailure::Unconfined and starts nothing — not that step, not
                  a later one. The check is independent of the model's: it holds for a Scenario value built in
                  code, and for a cwd that reaches outside through a symbolic link the fixture or an earlier step
                  made. Replay reports it as one Difference of Dimension::Malformed with subject "cwd"; record
                  returns RecordError::Malformed and writes nothing.
call capture      CALL_CAPTURE is declared with the runner and read by the record refusal and by replay. The fixture
                  builder is built before the runner and does not read it: until stub.rs exists the builder refuses
                  every scenario that has a [[stub]], unconditionally.
```

## Flows

```text
add a scenario for a behaviour the reference has
  1. write scenarios/<command path joined by "-">/<name>/scenario.toml and files/
  2. cargo dev record <id> --reference <path to the reference executable>
  3. read record/: the stored stdout, stderr, and state are what the binary must now equal
  4. cargo scenarios <id>
register a deviation
  1. the decision is recorded
  2. copy the affected files of record/steps/<nn>/ to expected/steps/<nn>/ and edit them to the decided behaviour
  3. add [deviation] decision = "<id>" to scenario.toml
re-record after the fixture format, git, or the pinned environment changes
  cargo dev record --rerecord <id>… ; the diff of record/ is reviewed like any other change
```
