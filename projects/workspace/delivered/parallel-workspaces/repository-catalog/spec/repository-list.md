# Repository list

The `repo list` command: which repositories it lists, in what order, and the document and table it writes. Its arguments
are the `repo list` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), Commands.
Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at `b57b020`.

## Interfaces

```text
# product:repository-roles, product:inventory-presence
# decision:parallel-workspaces/repository-catalog/repository-identities
# reference: modules/workspace/handlers/repo_handler.py:68-99,207-228; modules/workspace/repository_factory.py:57-133;
#            config/workspace.py:137-143
repo list [--json]
order      1. workspace root, then the load       configuration-model.md, Root discovery, then Load
           2. the implicit repositories, probed
           3. the drift warning                   drift-warning.md; written by the domain entry, before it returns
           4. the result on stdout                written by the command file through output
entries, in this order
  1. singletons    product  when <root>/product is a directory, symbolic links followed
                   harness  when <root>/context/harness/.git exists, symbolic links followed, file or directory
                   in that order; the workspace root is never listed here
  2. standalones   the loaded standalone entries, in merged order
  3. projects      the loaded project entries, in merged order; two entries with one resolved name are both listed
merged order       the shared file's entries, then the local file's, each in file order; an entry the load skips is
                   not listed (configuration-model.md, Layering and Repository entries)
configuration-model.md = ../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md
```

```text
locations
  singleton    product -> <root>/product         harness -> <root>/context/harness
  standalone   <root>/<rel>, rel = its path when not empty, else its resolved name
  project      <root>/projects/<resolved name>
normalised as Python's pathlib joins: repeated "/" collapsed, "." components dropped, a trailing "/" dropped;
nothing is resolved and no symbolic link is followed. A ".." component is kept as text: the load refuses it in a
standalone path but not in a name, so name = ".." gives <root>/projects/.. or <root>/.., written as such
```

```text
--json     one Shape A document, an array (../../../../delivered/parallel-workspaces/component-runtime/spec/
           output-formats.md, Shape per command), one object per entry in the order above:
  singleton   {"name": "product" | "harness", "type": "singleton", "path": <absolute location>}
  standalone  {"name": <resolved name>, "type": "standalone", "path": <absolute location>}
  project     {"name": <resolved name>, "type": "project", "main_path": <absolute location>, "pinned": <bool>}
  no other key: no url, main_branch, ref, layer, or presence. "pinned" is the loaded value, coerced by truthiness
  (configuration-model.md, Repository entries). The key is "type" here and "kind" in the repo add and repo remove
  documents, as in the reference tool.
  an empty inventory is []
human      one row per entry, in the same order, carrying:
             the source URL, or "-" when the entry has none (every singleton has none)
             "singleton" | "standalone" | "project"
             the location relative to <root>
             for a pinned project, a pinned mark
           layout is the binary's own (output-formats.md, Human output); nothing is written for an empty inventory
exit       0
effects    none: no file is created or written, no program is started, no setup command is run
```

Reference human form, an example under `decision:functional-parity`: a borderless table, columns two spaces apart, no
header row, trailing empty cells dropped (`core/internal/click_cli_output_service.py:35-132`).

```text
-                                  singleton   product
https://github.com/org/ctx.git     standalone  .winter/ext/context
https://github.com/org/api.git     project     projects/api       [pinned]
```

Failures: a missing root and every configuration fault end as the `ConfigError` of the section that defines it; the
listing of `projects/` fails as [drift-warning.md](./drift-warning.md) says. Nothing else fails.

## Domain

```rust
// crates/core/src/repository_catalog.rs
pub(crate) enum Listed {
    Singleton { name: &'static str, path: PathBuf },          // "product" | "harness"
    Standalone { name: String, url: Option<String>, path: PathBuf },
    Project { name: String, url: Option<String>, main_path: PathBuf, pinned: bool },
}
// The entries in the order of Interfaces. The drift notice, when there is drift, is written through `diagnostics`
// before the entries are returned, so it precedes the result on every path.
pub(crate) fn list(files: &dyn ReadFiles, start: &Path, diagnostics: &mut Diagnostics<'_>)
    -> Result<Vec<Listed>, ListError>;
pub(crate) enum ListError { Config(ConfigError), Internal(InternalError) }

// crates/core/src/diagnostics.rs — decision:parallel-workspaces/workspace-configuration/domain-reporting.
// Built with repo list, its first domain entry. The command file makes it from cx.err and cx.log and hands it to the
// entry; it is no port, and no Invocation or Context carries it.
pub(crate) struct Diagnostics<'a> { pub err: &'a mut dyn std::io::Write, pub log: Log }
```

The entries come from `workspace.config.standalone_repositories` and `workspace.config.project_repositories` of the
loaded `Workspace`
([discovery-and-loading.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/discovery-and-loading.md),
Domain): the resolved name is `common.name`, the URL `common.url`, a standalone location `<root>/<path>`, and `pinned`
as loaded. `Drift` is [drift-warning.md](./drift-warning.md), Domain. The entry writes the notice with
`diagnostics::notice` of [runtime.md](../../../../delivered/parallel-workspaces/component-runtime/spec/runtime.md),
Diagnostics on the stderr handle, which is built with this, its first caller.
