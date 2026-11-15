# Scenario format

What one recorded scenario is on disk: its directory, its description file, the fixture it describes, the record the
reference tool produced, and the expectation that replaces a record for a registered deviation. How the files are
produced and compared is [record-and-replay.md](./record-and-replay.md). The isolated tree, the pinned environment, and
the git helpers the fixture is built with are the
[test harness](../../../parallel-workspaces/component-runtime/spec/test-harness.md).

## System

```text
# engineering:scenario-files
scenarios/
  reference.toml                         the pinned reference: one per repository
  <command>/                             the command path joined by "-":  ws-index/   repo-remove/   doctor/   service-up/
    <name>/                              [a-z0-9][a-z0-9-]*          scenario id = "<command>/<name>"
      scenario.toml                      description: fixture, steps, comparison rules
      files/                             optional; copied into <ROOT>/ws before the workspace root commit
      dirty/                             optional; copied into <ROOT>/ws after every git step of the fixture
      record/                            written by the record step only
        meta.toml
        steps/<nn>/                      nn = 01, 02, … in step order
          exit-code                      decimal, one line; "signal:<n>" when killed by a signal
          stdout   stderr                raw bytes, ROOT-normalized
          stdout.json                    present when the step's stdout mode is json or json-lines
          before/  after/                state snapshots: tree.txt, git.txt, files/
          calls/<nnn>.json               present when the scenario stubs a program
      expected/                          optional; written by a person, with [deviation]
        steps/<nn>/…                     any subset of the files of record/steps/<nn>/
discovery     a scenario is every directory exactly two levels under scenarios/ ; a directory one level down with no
              child directory, a file beside reference.toml, or a name outside the pattern is malformed
```

```toml
# scenarios/reference.toml
commit = "b57b020e99e10a04d8bb621d0b03ddb2c983a090"     # the commit of the reference tool's checkout every record is made at
```

## Data

### scenario.toml

```toml
# engineering:scenario-files — unknown keys are an error at every level
summary = "one sentence: the behaviour this scenario pins"
needs = []                       # facilities; a non-empty list the runner cannot satisfy defers the scenario

[fixture]
workspace_git = true             # the workspace directory is a git repository with one commit; default false
gitignore = "…"                  # content of <ROOT>/ws/.gitignore when workspace_git; default: none written

[[fixture.remote]]               # a bare repository at <ROOT>/remotes/<name>.git
name = "alpha"
default_branch = "main"          # default "main"
[[fixture.remote.commit]]
message = "initial"
files = { "README.md" = "alpha\n" }
delete = []                      # optional
branch = "main"                  # optional; default the remote's default branch
tag = "v1"                       # optional

[[fixture.checkout]]             # a clone of a remote inside the workspace
remote = "alpha"
path = "projects/alpha"          # relative to <ROOT>/ws

[[fixture.worktree]]             # a worktree of a checkout
checkout = "projects/alpha"
path = "feature-one/alpha"
branch = "feature-one"
start = "origin/main"

[[fixture.git]]                  # a raw git call, after the entries above, in file order
cwd = "feature-one/alpha"        # relative to <ROOT>/ws
args = ["commit", "--allow-empty", "-m", "local"]

[[step]]                         # at least one
args = ["ws", "index", "feature-one", "--json"]
cwd = "."                        # relative to <ROOT>/ws; default "."
stdin = "stdin"                  # optional; a file of the scenario directory
stdout = "json"                  # "text" (default) | "json" | "json-lines"
env = { WINTER_LOG_LEVEL = "debug" }     # optional additions to the pinned environment; "<ROOT>" is expanded

[compare]
message_text = ["/warnings/*"]   # JSON pointers, "*" matching any one key or index
git_config_ignore = ["core.fsmonitor"]

[[compare.fan_out]]              # contiguous documents of these types, one partition per value at `key`
types = ["progress", "result"]
key = "/repo"

[[compare.file]]                 # a path rule; first match wins; unmatched paths compare as bytes
path = ".winter/state.toml"      # glob over ROOT-relative paths, "*" within a segment, "**" across segments
mode = "toml"                    # "bytes" | "toml" | "presence" | "ignore"
reason = "the writer's key order is not function"

[[stub]]                         # a recording program placed in <ROOT>/bin
name = "docker"
exit = 0
stdout = ""                      # optional text the stub prints
env = ["WINTER_ENV", "WINTER_WORKSPACE_ROOT"]      # variables the stub records

[deviation]                      # present exactly when expected/ exists
decision = "decision:parallel-workspaces/component-runtime/git-failure-rendering"
```

```text
workspace content
  .winter/config.toml and every other workspace file come from files/, verbatim
  files/ and dirty/ may hold the token <ROOT>; it is replaced with the real root when copied
build order
  remotes -> files/ -> checkouts -> worktrees -> workspace root commit (when workspace_git) -> [[fixture.git]] -> dirty/
references
  a [[fixture.checkout]] names a declared remote, and a [[fixture.worktree]] a declared checkout; an entry that
  names neither fails the build before any git call
no tool-made state      # decision:rust-adoption/parity-validation/scenario-state-prerequisites
  no key runs the reference tool or the binary before the first step; a step is always compared
stubs
  [[stub]] is parsed and validated by the model from the start. A fixture builder without call capture
  (runner::CALL_CAPTURE == false) refuses a scenario that has one: at record time the scenario is refused, at replay
  it fails with one Difference of Dimension::Malformed whose subject is "stub". It is never silently run without
  its stub.
```

### Path confinement

Every path a scenario names is used inside the scenario's own temporary tree, or the scenario is malformed.

```text
# engineering:record-step-separation, engineering:disposable-runtime-proof
key                                               base                      form
[[step]] cwd                                      <ROOT>/ws                 relative path
[[fixture.git]] cwd                               <ROOT>/ws                 relative path
[[fixture.checkout]] path                         <ROOT>/ws                 relative path
[[fixture.worktree]] checkout, path               <ROOT>/ws                 relative path
keys of [[fixture.remote.commit]] files; delete   the repository's root     relative path
entries under files/ and dirty/                   <ROOT>/ws                 relative path, by construction
[[step]] stdin                                    the scenario directory    relative path
[[fixture.remote]] name, [[stub]] name            —                         one segment: [A-Za-z0-9][A-Za-z0-9._-]*
```

```text
relative path — checked by the model when the description is parsed, before any tree exists
  not empty; holds no NUL; does not start with "/"; and, read segment by segment from the base, never stands
  above it: "." stays, ".." goes up one, any other segment goes down one, and no prefix of the path ends above
  the base
      accepted   "."   "feature-one/alpha"   "a/../b"
      malformed  "/tmp/x"   ".."   "../ws"   "a/../../b"
  [[step]] stdin, additionally: canonicalized, it is a regular file under the canonical scenario directory
arguments and environment — checked by the model likewise
  [[fixture.git]] args    args[0] matches [a-z][a-z0-9-]* : a git subcommand, never a global option such as -C, -c,
                          or --git-dir. No argument starts with "/" or "~", holds "=/" or "=~", or has ".." as a
                          path segment.
  [[step]] args           no argument starts with "/"
  [[step]] env            no value starts with "/": a value names a path only through <ROOT>
resolved — checked on the built tree, by whoever is about to use the path
  the fixture builder     before it creates, writes, or clones at a confined path, and before each git call it
                          makes with a confined cwd: the check of the test harness (test-harness.md, Interfaces,
                          "confinement"), the one check the builder and the git helpers share
  the runner              before each step: the step's cwd, canonicalized, is an existing directory under the
                          canonical <ROOT>          (record-and-replay.md, Runner, "confinement")
  links in the fixture    a symbolic link under files/ or dirty/ is copied as a link and never followed
```

```text
effect of a violation: the scenario is malformed, and the error names the scenario id and the key
  model check       discovery lists the scenario as malformed. No tree is created and no executable is started for
                    it — not git, not the reference tool, not the binary — at record or at replay.
  resolved check    the write or call that would have used the path is not made, and none follows it: neither the
                    reference tool nor the binary is started for that step or a later one.
  either            at replay the scenario fails with one Difference of Dimension::Malformed whose subject is the
                    key; at record it is refused and no record/ is written.
```

### Facilities

```text
# engineering:deferred-scenario-accounting
known to the format     pty   signal   clock   daemon   follow   provider   network   concurrency
supported by the runner the empty list, and every facility in runner::SUPPORTED (empty when the format lands)
effect                  needs ⊄ SUPPORTED -> the scenario is listed as deferred with the missing facilities; not run
                        a name outside the known list -> the scenario fails as malformed
stdin, stubs            are not facilities
```

| Facility      | Marks a scenario whose behaviour needs                                                                    |
| ------------- | --------------------------------------------------------------------------------------------------------- |
| `pty`         | a terminal on a standard stream: prompts read from a terminal, the dashboard, colour, progress rewriting. |
| `signal`      | a signal delivered to the process during the step.                                                        |
| `clock`       | a controlled time: timeouts, backoff, timestamps that are compared.                                       |
| `daemon`      | a process that outlives the step.                                                                         |
| `follow`      | output observed while the process is still running, such as `--follow`.                                   |
| `provider`    | a live service provider beyond a recording stub.                                                          |
| `network`     | a transport other than the fixture remotes.                                                               |
| `concurrency` | two invocations overlapping in time.                                                                      |

### record/meta.toml

```toml
format = 1
reference_commit = "<40 hex>"          # equals scenarios/reference.toml
git_version = "2.47.1"                 # the git the record ran with; informational
reference_runtime = "<one line>"       # first line the reference prints for --version; informational
environment = "<40 hex>"               # PinnedEnv::digest()
description = "<40 hex>"               # the description digest below
```

```text
description digest    SHA-1 over, in this order: the bytes of scenario.toml; then for every regular file under files/,
                      then dirty/, then the stdin files the steps name, sorted by scenario-relative path bytes:
                      the path, a NUL byte, the content length as decimal, a NUL byte, the content
```

### State snapshot

```text
# engineering:behavioral-fixtures — taken before the first step and after every step, over the whole ROOT
tree.txt        one line per entry under <ROOT>, sorted by path bytes, paths ROOT-relative
                  d <path>
                  f <x|-> <sha1 of content> <path>          x = any execute bit set
                  l <path> -> <target, ROOT-normalized>
                the hash is of the content as files/<path> stores it: ROOT-normalized when the content is valid
                UTF-8, the raw bytes otherwise — so a file that holds the root path hashes equally under two roots
                excluded: the excluded roots below, the interior of every git directory,
                and __pycache__ directories with their content
files/<path>    the content of every regular file that tree.txt lists and that differs from, or is absent in,
                the previous snapshot of the same scenario; before/ of step 01 stores none
                content is ROOT-normalized when it is valid UTF-8
                <path> is the tree.txt path in its stored form (stored paths, below)
git.txt         one section per repository, sections sorted by path; a repository is any directory under <ROOT>
                that holds a .git entry, plus every remotes/*.git
excluded roots  the entries bin/  calls/  git-template/  cache/  tmp/  of <ROOT> itself are excluded from the
                snapshot as a whole: nothing at or beneath them is listed in tree.txt or stored under files/, and
                a repository beneath them has no git.txt section
reading         a snapshot is read together with the previous one of the same scenario. An f line implies a file
                under files/ when the previous tree.txt has no f line for its path, or has one with another hash;
                a line that differs in the execute flag alone implies none, and before/ of step 01 implies none.
                Reading fails, naming the path, when an implied file is absent; when the content of a stored file
                does not hash to the value on its tree.txt line; and when files/ holds a file that no f line
                implies — a path without an f line, a hash unchanged from the previous snapshot, or any file
                where there is no previous snapshot. A record that lost or changed stored content never reads as
                complete.
                tree.txt and git.txt are regular files, and files, when it exists, is a directory; a link in the
                place of any of the three is not followed and fails the read, naming the entry.
                Under an expected/ overlay these rules judge the overlaid whole against the overlaid tree.txt:
                the file taken for an implied path is hashed, a file under expected/ that nothing implies fails
                the read, and a file under record/ that the overlaid tree.txt does not imply is left out.
```

```text
stored paths — every files/<path> of a snapshot, under record/ and under expected/. A record is committed to the
repository that holds the scenarios, and no stored segment may be special to that repository.
  written       segment by segment: every "%" becomes "%25"; then a segment that began with the four bytes ".git"
                has its leading "." written as "%2E". No other segment and no other byte changes.
                    ws/feature-one/alpha/.git    ->   files/ws/feature-one/alpha/%2Egit
                    ws/.gitignore                ->   files/ws/%2Egitignore
                    ws/docs/.gitattributes       ->   files/ws/docs/%2Egitattributes
                    ws/50%.txt                   ->   files/ws/50%25.txt
                    ws/%2Egit                    ->   files/ws/%252Egit
                    ws/.github/x   ws/a.git/x    ->   files/ws/%2Egithub/x   files/ws/a.git/x
  read          exactly the reverse, segment by segment: a leading "%2Egit" becomes ".git"; then every "%25"
                becomes "%". A segment that begins with ".git", or holds a "%" that neither rule accounts for,
                is not a stored form and fails the read.
  elsewhere     tree.txt, [[compare.file]] path, and the subject of a Difference hold the tree's real path
```

```text
git.txt section, each part a labelled block; every path ROOT-normalized
  repository      <ROOT-relative path>            kind: worktree | main | bare
  dot-git         "directory"; the gitdir: line of the .git file; or, when .git is a symbolic link,
                  "link -> <target>", the target ROOT-normalized like any link target      (not bare)
  head            `git rev-parse HEAD` or "unborn";  `git symbolic-ref -q HEAD` or "detached"
  refs            `git for-each-ref --format=%(refname) %(objectname) %(upstream) %(symref)`
  worktrees       `git worktree list --porcelain`             (main and bare only)
  status          `git status --porcelain=v2 --branch --untracked-files=all`    (not bare)
  stash           `git stash list --format=%gd %H`                      (not bare)
  config          `git config --local --list`, sorted, minus [compare] git_config_ignore
  in-progress     which of MERGE_HEAD, REBASE_HEAD, rebase-merge/, rebase-apply/, CHERRY_PICK_HEAD,
                  REVERT_HEAD, BISECT_LOG exist
  exclude         content of info/exclude
  log             per local branch: `git log --format=%H %P %an %ae %at %cn %ce %ct %s`
bare              a bare repository has no .git entry and no work tree, so its section has no dot-git, status, or
                  stash block; a refs/stash it holds still shows in refs
excluded          reflogs, the index file, ORIG_HEAD, FETCH_HEAD, packed-refs layout, object storage layout, hooks
```

### Normalization

```text
# engineering:pinned-scenario-environment
ROOT            every occurrence of the canonical root path, as bytes, becomes the six bytes <ROOT>, in stdout,
                stderr, file content, link targets, git output, and call records; applied before JSON is parsed
REFERENCE       at record only: every occurrence of the canonical path of the reference tool's checkout, as bytes,
                becomes <REFERENCE>, after ROOT and in the same places, so a path the reference tool prints about
                itself — in a failure trace, say — is never stored. The recorder is handed the path as a redaction
                (record-and-replay.md, Record). Replay applies none: the binary has no such path to print, and a
                <REFERENCE> inside a compared value fails replay like any other value the binary does not produce.
JSON            stdout.json holds the parsed stream re-encoded: one value per line, keys sorted, no insignificant
                whitespace; under json-lines a line that is not JSON is stored as {"$text": "<line>"}
nothing else    object ids, dates, and ports are made equal by the pinned environment and the fixture,
                not by rewriting; a scenario whose record is unstable between two roots
                (record-and-replay.md, Record) is not stored
```

### Program calls

```text
calls/<nnn>.json     one per stub invocation, numbered in start order
  { "program": "docker", "argv": ["compose", "up", "-d"], "cwd": "<ROOT>/ws", "env": { "WINTER_ENV": "feature-one" },
    "stdin_bytes": 0 }
env                  exactly the variables the [[stub]] entry lists; an unset one is absent
```

### Expectation for a registered deviation

```text
expected/steps/<nn>/<file>     replaces record/steps/<nn>/<file> for comparison, file by file
                               after/files/<path> and after/tree.txt are replaced together or not at all
                               <path> is in its stored form, and the overlaid snapshot is read under "reading" of
                               State snapshot: an implied file is taken from expected/, else from record/
[deviation].decision           the id of the decision that registers the difference
consistency                    expected/ without [deviation], or [deviation] without expected/, is malformed
the record step                never reads or writes expected/
```

## Open questions

- The stub program is a shell script the fixture builder writes into `<ROOT>/bin`; whether any stubbed program must
  answer differently per call is decided when the first service scenario needs it.
