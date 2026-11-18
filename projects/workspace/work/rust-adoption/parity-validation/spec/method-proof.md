# Method proof

The two behaviours the scenario method is proven on, the scenarios that pin them, and the part of the binary that exists
once they pass. The method is [scenario-format.md](./scenario-format.md) and
[record-and-replay.md](./record-and-replay.md). Each behaviour's contract is owned by the epic that owns the command;
this leaf links it and states only which scenarios pin it and how far the proof implements it.

## System

```text
# engineering:method-proof, decision:rust-adoption/parity-validation/proof-behaviours
read-only         ws index NAME --json
                  contract: ../../../parallel-workspaces/environment-lifecycle/spec/index-lookup.md
state-changing    repo remove TARGET [--local] --json
                  contract: ../../../parallel-workspaces/repository-catalog/spec/repository-removal.md
```

```text
crates/core/src/                what the proof adds; each module's contract is its owner's spec
  commands/ws/index.rs          the command file                                    index-lookup.md
  commands/repo/remove.rs       the command file                                    repository-removal.md
  workspace_root.rs             the workspace root of a directory                   configuration-model.md, Root discovery
  config_raw.rs                 env_aliases and envs_per_workspace, layered         configuration-model.md, Load, Layering
  state_registry.rs             the env index registry, read only                   lock-and-state.md, `.winter/state.toml`
  env_index.rs                  lookup and the suggested index                      lock-and-state.md, Env index allocation
  config_edit.rs                removal of one repository block from TOML text      repository-removal.md
  files.rs                      the file port: ReadFiles with ws index, WriteFiles with repo remove
                                ../../../parallel-workspaces/component-runtime/spec/repository-layout.md,
                                What a command receives
configuration-model.md and lock-and-state.md are those of ../../../../delivered/parallel-workspaces/workspace-configuration/spec/
```

### How far the proof implements an owner's contract

| Contract                                               | Implemented by the proof                                                                                                                             | Left to the owner's own work                                                  |
| ------------------------------------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| configuration-model.md, Root discovery                 | whole                                                                                                                                                | —                                                                             |
| configuration-model.md, Load, Layering, Top-level keys | for `env_aliases` and `envs_per_workspace` only: read both files, parse, Replace layering, default, coercion, and the cross-key rule between the two | every other key and table, unknown and removed keys, notices, the typed model |
| lock-and-state.md, `.winter/state.toml`                | reading the registry, with that section's cases for an absent or malformed file                                                                      | every write, the rest of the file                                             |
| lock-and-state.md, Env index allocation                | the layout, `suggested`, and `lookup`: registry first, else the alias position, else the hash slot, with no probe                                    | `allocate` with its probe loop, `release`, `EnvIndexError`, Band exhaustion   |
| configuration-model.md, Write integrity                | whole, for one file, as the `WriteFiles` implementation: the lock, the replacement, kept permission bits, and the symlink rule                       | `config.lock` and `state.toml` writers                                        |
| index-lookup.md                                        | whole                                                                                                                                                | —                                                                             |
| repository-removal.md                                  | whole                                                                                                                                                | every other `repo` command                                                    |

The owning epic's later work extends these modules and does not replace them. A fault the partial implementation can
meet but that this table leaves with the owner — an unknown key, say — is not detected by the proof.

## Interfaces

### Proof scenarios

Every fixture is files only: no remote, no checkout, no worktree, no facility, no stub. Paths are relative to the
scenario's `files/`.

| Scenario id                        | Fixture                                                                                 | Step                                     | Pins                                                                            |
| ---------------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------- | ------------------------------------------------------------------------------- |
| `ws-index/alias-suggested`         | `.winter/config.toml` with `env_aliases = ["one", "two"]` and `envs_per_workspace = 12` | `ws index two --json`                    | keys are read; alias index; `source` `suggested`; nothing written               |
| `ws-index/hashed-suggested`        | `.winter/config.toml` holding one comment line                                          | `ws index feature-x --json`              | defaults; the allocation formula (lock-and-state.md, Fixed points)              |
| `ws-index/registry-recorded`       | the same, plus `.winter/state.toml` recording `feature-x` at 15                         | `ws index feature-x --json`              | registry wins; `source` `registry`                                              |
| `ws-index/suggested-slot-taken`    | as `hashed-suggested`, plus `.winter/state.toml` recording `other` at 40                | `ws index feature-x --json`              | lookup does not probe: `index` 40, `source` `suggested`, though 40 is `other`'s |
| `repo-remove/removes-last-block`   | `workspace_git = true`; the plain configuration below                                   | `repo remove project/docs --json`        | the file's bytes; `git status` of the root shows the one file                   |
| `repo-remove/removes-middle-block` | `workspace_git = true`; the commented configuration below; a registered deviation       | `repo remove project/web --json`         | which comment and blank lines go with a block; match by URL name                |
| `repo-remove/not-found`            | as `removes-middle-block`, without the deviation                                        | `repo remove project/nope --json`        | exit 1; every file unchanged                                                    |
| `repo-remove/local-file-absent`    | as `not-found`                                                                          | `repo remove project/api --local --json` | exit 1; no `.winter/config.local.toml` is created                               |

```text
suggested-slot-taken
  40 is the `feature-x` row of lock-and-state.md, Fixed points, under the default configuration, so the fixture
  records another name at exactly the slot the looked-up name hashes to. An implementation that probed past a
  taken slot would answer 41 and fail replay; hashed-suggested and registry-recorded pass either way.
  files/.winter/state.toml, whole:
      [env_index]
      other = 40
```

```toml
# the plain configuration: files/.winter/config.toml of removes-last-block, byte for byte
# workspace repositories

[[project_repository]]
url = "https://fixture.invalid/api.git"

[[project_repository]]
name = "docs"
url = "https://fixture.invalid/docs.git"
```

```toml
# the commented configuration: files/.winter/config.toml of the other three repo-remove scenarios, byte for byte
# workspace repositories

[[project_repository]]
url = "https://fixture.invalid/api.git"
# after api

# second: web
[[project_repository]]
url = "https://fixture.invalid/web.git"
# after web

# third: docs
[[project_repository]]
name = "docs"
url = "https://fixture.invalid/docs.git"
```

The first line of each block above names the block and is not part of the file; the file starts at
`# workspace repositories`.

```text
why two configurations
  in the commented one every block has a comment directly above it or directly above the next block, so every
  successful removal from it falls under the registered deviation. The plain one has no comment line next to a
  repository header: removing its last block is the same in both tools — the docs header to the end of the file
  goes, and the file then ends with the api url line and one blank line — so removes-last-block replays against
  its record alone and proves the byte comparison without an expectation.
```

### The deviation scenario

`repo-remove/removes-middle-block` is where the binary differs from the reference tool on purpose, under
`decision:parallel-workspaces/repository-catalog/removal-comment-extent`. It is expressed as
[scenario-format.md](./scenario-format.md), Expectation for a registered deviation, expresses one. It shows both halves
of the decision at once: `# second: web` is the removed block's own comment, and `# third: docs` is the next block's.

```text
record/                      what the reference tool did, recorded like any other scenario and never edited
  predicted from repository-removal.md's reference column: the file loses web's header and url lines,
  "# after web", the blank line, and "# third: docs"; "# second: web" stays and then stands directly above the
  docs block
expected/steps/01/after/
  files/ws/.winter/config.toml     the text below, byte for byte
  tree.txt                         the record's after/tree.txt with one change: the hash on the line of
                                   ws/.winter/config.toml is the SHA-1 of the text below
scenario.toml
  [deviation]
  decision = "decision:parallel-workspaces/repository-catalog/removal-comment-extent"
every other file of the step — exit-code, stdout.json, after/git.txt — is compared against record/ as usual
```

```toml
# workspace repositories

[[project_repository]]
url = "https://fixture.invalid/api.git"
# after api

# third: docs
[[project_repository]]
name = "docs"
url = "https://fixture.invalid/docs.git"
```

The block above is `expected/steps/01/after/files/ws/.winter/config.toml` whole, byte for byte. Against the commented
configuration it lacks five lines: `# second: web`, web's header and url lines, `# after web`, and the blank line after
it. The blank line between `# after api` and `# second: web` is before the removed range and stays.

```text
the record still decides what the reference does. When record/ differs from the prediction above, the reference
column of repository-removal.md is corrected to the record; the expectation changes only if the corrected column
changes which lines the decision speaks about.
the expectation differs from the predicted record in exactly one line: where the record has "# second: web", the
expectation has "# third: docs". A wider difference means the removal rule or the expectation is wrong.
```

### Proof that a wrong implementation is caught

```text
# engineering:method-proof — crates/test-support/tests/proof.rs, part of `cargo test`, no reference tool involved
for a copy of a recorded proof scenario in a temporary directory, replayed with a stand-in executable
(a shell script the test writes) that:
  ws-index/hashed-suggested
    prints the recorded stdout and exits with another code       -> exactly one Difference, Dimension::Exit
    prints stdout with one JSON value changed                    -> Dimension::Json, subject = the pointer
    reproduces the record exactly                                -> no Difference
  ws-index/suggested-slot-taken
    prints the recorded document with "index" one higher         -> Dimension::Json, subject = /index:
                                                                    a lookup that probes fails replay
  repo-remove/removes-last-block
    leaves the configuration file with one byte changed          -> Dimension::Files and Dimension::Tree
    reproduces the record exactly                                -> no Difference
  repo-remove/removes-middle-block
    writes the configuration file of expected/                   -> no Difference
    writes the configuration file of record/                     -> Dimension::Files and Dimension::Tree:
                                                                    the reference's own behaviour fails replay
```
