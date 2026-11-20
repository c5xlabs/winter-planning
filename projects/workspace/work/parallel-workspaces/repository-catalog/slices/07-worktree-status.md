---
name: worktree-status
depends-on:
  - worktrees-command
code:
  - winter-workspace/crates/core/src/context.rs::Context
  - winter-workspace/crates/core/src/process.rs::run_with
  - winter-workspace/crates/core/src/termination.rs::Termination
  - winter-workspace/crates/dev/src/architecture.rs::FORBIDDEN_TOKENS
---

# `ws worktrees --status`: the program port and the git probes

Adds `--status` and the probes of [location-listing.md](../spec/location-listing.md), With `--status`, together with the
port they are the first to call, [child-processes.md](../spec/child-processes.md) apart from its spans. The main branch
of each probe is the loaded `common.main_branch`, which already falls back to the workspace's. Delivers the Git-details
bullet of `product:inventory-presence`, the main-branch override bullet of `product:repository-addresses`, and
`engineering:catalog-effect-separation`. A failed worktree probe ends as `Termination::Repo`, written once under
`decision:parallel-workspaces/component-runtime/git-failure-rendering`. Spans follow in `probe-spans`, under
`decision:parallel-workspaces/repository-catalog/direct-landing`.

## Change

- `crates/core/src/programs.rs` holds `Program`, `Call`, `Finished`, `Exit`, `RunPrograms`, `OsPrograms`, and
  `find_program` as the spec's Domain gives them. `OsPrograms::new` builds the child environment once from the snapshot:
  every name and value as given; `WINTER_LAUNCH_TIME` left out; `GIT_SSH_COMMAND` set to the spec's default when absent,
  an empty value kept; `LANGUAGE` and `LC_ALL` set to `C` for git; `TRACEPARENT` and `TRACESTATE` as the snapshot holds
  them. `run` finds `git` with `find_program` on the snapshot's `PATH` (`/bin:/usr/bin` when unset), starts it with the
  call's arguments and working directory, stdin from the null device, stdout and stderr captured, and waits with no
  timeout; a program that cannot be started is `Err`, which the caller ends as `InternalError::Unexpected`.
- `ProcessEnv` gains `pairs`, as [child-processes.md](../spec/child-processes.md), Domain, gives it; `OsPrograms::new`
  reads the snapshot through it.
- `Context` gains `programs`, and `run_with` builds one `OsPrograms` from the invocation's snapshot beside `OsFiles`.
  `changes.md`, A command, states that the one call is also handed `cx.programs` when it starts a program.
- `ReadFiles` gains `read_link` on `OsFiles` and `MemoryFiles`, as [drift-warning.md](../spec/drift-warning.md), Domain,
  gives it; the repository rule reads the target of a `HEAD` link through it.
- `Termination::Repo(Box<RepoError>)`, `RepoError`, and the git-failure variant of `RepoMessage` are added as
  [results-and-exit-codes.md](../../../../delivered/parallel-workspaces/component-runtime/spec/results-and-exit-codes.md),
  Domain and Error rendering, give them, with `"Repo"` added to `Termination::error_type` and the rows
  `context/architecture/changes.md` requires for a new variant.
- `--status` is registered. For each environment present, in order, every loaded project entry is probed, whether or not
  its location exists: a location that is no repository, by the spec's rule, gives 0, 0, 0 with no git call; otherwise
  `git status --porcelain=v2 --branch --untracked-files=all -z` gives dirty as the spec counts it, then
  `git rev-list --left-right --count origin/<main>...HEAD` gives behind and ahead, with the
  `git rev-parse --verify --quiet origin/<main>` fallback giving 0, 0 or the `RepoError`; `main` is the entry's
  `common.main_branch` as loaded. A failed git call writes the record of location-listing.md, Failures, through the
  `Diagnostics` that `locations::list` now takes; the command file builds it from `cx.err` and `cx.log`. Every probe of
  an environment runs; the first failure in project order then ends the command, and no later environment is probed. Two
  entries with one resolved name both show the later entry's counts.
- Standalones get null counts and no git call. The workspace probe makes the seven calls of location-listing.md, With
  `--status`, in their order, and gives null counts where that block says; else dirty from the staged, unstaged, and
  untracked paths, and ahead and behind against the upstream operand, 0, 0 without one.
- `Location` gains `status` and `ListError` gains `Repo`. The document gains `ahead`, `behind`, and `dirty` on every
  entry, and the human rows the counts.
- The architecture check forbids `process::Command` in the library outside `programs.rs`, with the pinned test of the
  token list updated; `RunPrograms` and `OsPrograms` join `COMMAND_FILE_NAMES` beside `ReadFiles` and `OsFiles`, its
  pinned test renamed for the new count, and not the module name, which `cx.programs` would match;
  `context/architecture/structure.md` and `changes.md` gain the `programs.rs` row.
- The open question of location-listing.md on how the workspace probe counts is answered by the scenarios recorded here;
  where a record differs from the spec's calls, the calls follow the record, and the spec is corrected from it in a
  change to the planning repository.
- Recorded scenarios under `scenarios/ws-worktrees/`, recorded with `cargo dev record` from the reference tool, the
  executable path given with the task, with `--status --json`: worktrees ahead of, behind, and level with their remote's
  main branch; a project with `main_branch = "dev"` and a workspace with top-level `main_branch = "trunk"`, each remote
  holding both branches; a worktree with a staged, an unstaged, and an untracked file, and an untracked directory of two
  files; a project whose remote has no main branch fetched; a worktree location that is a plain directory; a worktree
  location holding a bare repository made by the raw git call `git init --bare`, which the spec makes a `RepoError`; the
  workspace root on a branch with an upstream, with none, with a local upstream (`branch.<b>.remote = .`), and detached;
  at the root, a staged rename, and a file staged by a raw `git add` and then changed by `dirty/`; a standalone.

## Acceptance

- `cargo scenarios ws-worktrees` passes every scenario; the bare-repository record shows exit 1 and nothing on stdout;
  the `main_branch` records count against `origin/dev` and `origin/trunk`.
- Unit tests of `locations::list` with a `RunPrograms` of the test's own: a `git status` exiting 128 ends as `RepoError`
  with the program, subcommand, arguments, cwd, exit, and stderr of the call; with two environments whose first fails,
  every probe of the first runs and none of the second; a failure in the workspace probe gives nulls; an `Err` from the
  port is an internal failure; without `--status` no call is made; a failure in the workspace probe writes no record;
  for an unborn HEAD the staged call is not made; a missing `branch.<b>.merge` gives 0, 0 with no `rev-list` call.
- Unit tests of `read_link` inside a `TempTree`: a link gives its target as written, relative or absolute; a file, a
  directory, and a missing path give `None`. The same cases on `MemoryFiles`.
- Unit tests of `ProcessEnv::pairs`: a name and a value that are not UTF-8 come back as given.
- Unit tests of `find_program`: the first entry holding an executable wins; an entry whose file is not executable is
  skipped; an empty entry is the working directory; `PATH` unset searches `/bin` and then `/usr/bin`.
- Unit tests of the repository rule with `MemoryFiles`: each case of the spec's rule, a `HEAD` link whose target starts
  with `refs` and one whose target does not, and an empty `.git` directory and a `.git` file without a `gitdir` line as
  none.
- Component tests of `ws worktrees --status` in the isolated tree, with a stub `git` first on the pinned `PATH` that
  writes its environment, its arguments, its working directory, and what it reads from stdin to `calls/`: the child sees
  a variable of the snapshot and not `WINTER_LAUNCH_TIME`; `GIT_SSH_COMMAND` is the default when the snapshot lacks it
  and the snapshot's value, `""` included, when it has it; `LANGUAGE` and `LC_ALL` are `C`; stdin reads empty. A stub
  exiting 128 gives exit 1, the failure written once on stderr, and nothing on stdout; with `-v` the same run writes it
  twice, once as the log record and once in the ending.
- An executable test with tracing on and the stub `git`: the child's environment holds the generic
  `OTEL_EXPORTER_OTLP_*` variable the test set, which the binary removed from its own environment.
- `cargo test -p winter-workspace-dev` passes, and a test of the architecture check shows `process::Command` in a
  library file other than `programs.rs` fails it while `files.rs`'s `std::process::id` passes.
- `cargo dev gate full` exits 0.

## Out of scope

- Operation spans: `probe-spans`. Remote calls and their timeout: the epics that fetch and push.
