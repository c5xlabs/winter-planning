---
name: repository-entries
depends-on:
  - load-seam
---

# Repository entries are read and refused at load

Fills `config_model/repositories.rs` with Repository entries, Value types, and the repository rows of Cross-key rules of
[configuration-model.md](../spec/configuration-model.md), under
`decision:parallel-workspaces/workspace-configuration/element-type-validation` and
`decision:parallel-workspaces/workspace-configuration/unhandled-reference-failures`. Serves
`product:configuration-validation` and `product:effective-configuration`.

## Change

- Both arrays are read into `ProjectRepository` and `StandaloneRepository` values in the order of the spec's Repository
  entries block: an element that is not a table is skipped; `name`, then `url`, present and not a string is refused; an
  element whose `name` and `url` are each absent or an empty string is skipped whole and no other field of it is read;
  then the name derived from the URL, every field rule of the table, `DuplicateStandaloneName`, and every `Repository`
  fault.
- A value that is not a string for `name`, `url`, `main_branch`, `ref`, `path`, `prefix`, or `config_dir`, the values
  `false`, `0`, `0.0`, `[]`, and `{}` included, and a value that is not an array of strings for `git_excludes` or `cmd`,
  is `ValueType` carrying the repository. The repository's label is its name, else its url, else `#<n>`, as the spec
  states. The values are read through the shared readers of `config_model.rs`.
- Recorded scenarios, each by `cargo dev record <id> --reference <path>` over a fixture workspace in a temporary
  directory:
  - plain: `ws-index/duplicate-standalone-name`, `ws-index/standalone-nested`, `ws-index/envs-without-nested`,
    `ws-index/standalone-path-escapes`, `ws-index/duplicate-project-name-accepted`,
    `ws-index/local-repository-appended`;
  - probes, recorded plain: `ws-index/repository-name-zero-with-url`, whose one project entry holds `name = 0` and a
    url, and `ws-index/repository-main-branch-zero`, whose one project entry holds a name and `main_branch = 0`;
  - deviations, each given
    `[deviation] decision = "decision:parallel-workspaces/workspace-configuration/element-type-validation"` and an
    `expected/` tree written by hand that holds exit code 1 and the empty answer a failing `--json` run stores, then
    recorded again with `--rerecord`: `ws-index/repository-cmd-string`, and `ws-index/repository-name-zero-without-url`,
    whose one project entry holds `name = 0` and no url.
- Files touched: `crates/core/src/config_model/repositories.rs`; `scenarios/ws-index/**` for the ten scenarios.

## Acceptance

- Unit tests assert the variant and carried values of each `RepositoryFault`, of `DuplicateStandaloneName` for a clash
  by name and a clash by URL, and of `ValueType` for each of the nine fields.
- A unit test table asserts `ValueType` for `name` set to each of `false`, `0`, `0.0`, `[]`, and `{}`, with and without
  a url, and the label `#1` when the entry has no url; and for `main_branch = 0` and `prefix = false` on a named entry.
- A unit test asserts what is skipped with no error: an element that is a string; an entry with neither `name` nor
  `url`; an entry with `name = ""`, `url = ""`, and `main_branch = 0`.
- A unit test table asserts the derived name for URLs ending in `.git`, in `/`, in `.git/`, and for an `scp`-style URL
  whose last separator is `:`.
- A unit test asserts that a local entry follows the shared entries and that a local entry with the name of a shared
  standalone is refused.
- `cargo scenarios ws-index` passes: the six plain scenarios and the two probes against the record of the reference
  tool, the two deviation scenarios against their `expected/` trees.
- The record of each probe shows the reference tool exiting 1, and the record of `repository-name-zero-without-url`
  shows it exiting 0; the change reports the three records. A probe whose record shows exit code 0 becomes a deviation
  scenario of the same decision, with an `expected/` tree as above, and the change reports it as a further case the
  reference tool accepts.
- `cargo dev gate slice` passes; no file outside those named changes.

## Out of scope

- A repository key whose value is not an array, which the load refuses before this child runs.
- The singletons and the catalog commands.
