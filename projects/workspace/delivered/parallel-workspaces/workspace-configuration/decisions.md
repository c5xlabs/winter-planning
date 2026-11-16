# Workspace configuration decisions

## Write byte preservation (decision:parallel-workspaces/workspace-configuration/write-byte-preservation)

**Decider:** product owner

A command that edits a configuration file keeps every byte outside its edit, line endings and encoding included, so a
file with CRLF line endings keeps them. The reference tool rewrites every line ending as its platform's and reads and
writes in the encoding of its locale, so the difference is a registered deviation.

## Repository array type (decision:parallel-workspaces/workspace-configuration/repository-array-type)

**Decider:** product owner

A `project_repository` or `standalone_repository` value that is not an array, in the shared or in the local file, is a
configuration error naming the key and the file, exit 1. The reference tool lets a local value that is not an array
replace the shared array, reads a string or a table as no repositories, and fails unhandled on a non-zero number, so
refusing all of them is a registered deviation.

## Configuration inspection (decision:parallel-workspaces/workspace-configuration/configuration-inspection)

**Open.** Whether the binary gains a command that explains the effective configuration and where each value comes from.
The reference tool has no such command. The product owner has deferred the question to the refinement of
`epic:agent-ready-operation/context-navigation`, which designs the `context` command. It blocks
`product:configuration-explanation`, which stays proposed, and nothing else in this epic.

## State read failures (decision:parallel-workspaces/workspace-configuration/state-read-failures)

**Decider:** product owner

A `state.toml` that exists and cannot be read, or is not UTF-8, or does not parse as TOML, is a configuration error
naming the file, exit 1, and never an empty registry. One rule covers every text that does not parse: the binary does
not ask why its parser refuses it. A path at which nothing exists is still an empty registry, and a file that parses is
read as the lock and state spec states. The reference tool ends with an unhandled error, exit 1, on a file it cannot
read, on one that is not UTF-8, and on a key given twice in the body of a table or inside an inline table. It reads a
syntax error, a top-level key given twice, and a table header given twice as an empty registry, so refusing those is a
registered deviation, which covers every text that does not parse and that the reference tool reads as empty. The lock
and state spec lists the cases.

## Replacement edges (decision:parallel-workspaces/workspace-configuration/replacement-edges)

**Decider:** product owner

A workspace file is always replaced by a temporary file renamed over it, under the lock of
`decision:parallel-workspaces/workspace-configuration/write-conflicts`; the binary never falls back to a write in place.
Where that can be told from the reference tool's write in place, the difference is a registered deviation: a second hard
link of the file keeps the old content; the owner and group of the file are not kept; a writable file in a directory
that takes no new file is refused; a symbolic link to a missing file is refused; a directory whose file system refuses
an advisory lock is refused; a `.winter/` directory that can be written and cannot be read is refused, the file left
unchanged; a file that is a mount point is refused when the system refuses the rename over it; a writer that is killed
leaves its temporary file beside the file, under
`decision:parallel-workspaces/workspace-configuration/abandoned-temporary-files`; and a writer waits, with no limit,
while another holds the lock. The write path spec states each as it is built.

## TOML dialect (decision:parallel-workspaces/workspace-configuration/toml-dialect)

**Decider:** product owner

The binary accepts and refuses TOML text as its parser does, in each of the four workspace files, and does not check the
text first so that the two tools agree. Where its parser and the reference tool's read a text differently, the
difference is one registered deviation; the discovery and loading spec lists the texts as a probe of the parser finds
them.

## Inline registry table (decision:parallel-workspaces/workspace-configuration/inline-registry-table)

**Open.** Whether an allocation keeps the entries of an `env_index` written as an inline table, or drops them as the
reference tool does. The product owner has deferred the question to the refinement of
`epic:parallel-workspaces/environment-lifecycle`, which builds the allocation write. It blocks that write alone and
nothing in this epic.

## Unhandled reference failures (decision:parallel-workspaces/workspace-configuration/unhandled-reference-failures)

**Decider:** engineering

Every configuration on which the reference tool ends with an unhandled error is a configuration error, exit 1, naming
the file or the key. The ending and the exit code are the reference tool's; the message is the binary's own.

## Loaded workspace value (decision:parallel-workspaces/workspace-configuration/loaded-workspace-value)

**Decider:** engineering

The root and the validated configuration are one value a domain entry loads through the read port and passes down;
`Context` and `Invocation` gain no field and nothing caches the value. Its types are part of the command contract and
may hold fields no command reads yet.

## Domain reporting (decision:parallel-workspaces/workspace-configuration/domain-reporting)

**Decider:** engineering

Loading the configuration writes no log record and no notice, as in the reference tool. A domain entry that reports
borrows the error stream and the log from its command file and is built with the first entry that needs it; it is no
port of the host.

## Unbuilt readers and writers (decision:parallel-workspaces/workspace-configuration/unbuilt-readers-and-writers)

**Decider:** engineering

The lock file reader and writer, the state file writes, and the configuration writers other than the removal of a
repository entry are specified in this epic and built by the epic whose command first calls them.

## Write failure kind (decision:parallel-workspaces/workspace-configuration/write-failure-kind)

**Decider:** engineering

A configuration file that cannot be replaced is a configuration error of a kind of its own, naming the file and the
cause, not a read failure.

## Abandoned temporary files (decision:parallel-workspaces/workspace-configuration/abandoned-temporary-files)

**Decider:** engineering

A temporary file left beside a configuration file by a writer that was killed is never read and never removed by the
binary.

## Non-regular configuration path (decision:parallel-workspaces/workspace-configuration/non-regular-configuration-path)

**Decider:** engineering

A local configuration path that exists and is not a regular file is read as a missing file, as in the reference tool.

## Element type validation (decision:parallel-workspaces/workspace-configuration/element-type-validation)

**Decider:** product owner

A value that is not an array of strings for `git_excludes`, or for a repository entry's `git_excludes` or `cmd`, and a
value that is not a string for a string field of a repository entry (`name`, `url`, `main_branch`, `ref`, `path`,
`prefix`, `config_dir`), the values `false`, `0`, `0.0`, `[]`, and `{}` included, is a configuration error naming the
key, exit 1; so is a shared array, an array of pairs included, where a table is expected and the local file holds the
same key. List keys with a coercion rule of their own, such as `env_aliases` and `capabilities.<slot>`, keep that rule.
The reference tool reads a bare string for `git_excludes` or `cmd` as one entry per character and a table as its keys,
skips a repository entry whose `name` and `url` are both empty whatever their types, and reads a shared array of pairs
as a table, so refusing all of them is a registered deviation.

## State file compatibility (decision:parallel-workspaces/workspace-configuration/state-file-compatibility)

**Decider:** engineering

`.winter/state.toml` and `.winter/config.lock` are read and written in the reference tool's formats. The refusal of a
`state.toml` that cannot be read or does not parse is decided by
`decision:parallel-workspaces/workspace-configuration/state-read-failures`.

## Root selection (decision:parallel-workspaces/workspace-configuration/root-selection)

**Decider:** product owner and engineering

The root is the nearest ancestor of the working directory that holds the file `.winter/config.toml`. There is no root
option or root environment variable, and no root found is a configuration error.

## Layering policy (decision:parallel-workspaces/workspace-configuration/layering-policy)

**Decider:** product owner and engineering

The local file overlays the shared file as the reference tool does: the two repository arrays append, seven named tables
merge one level deep, and every other key is replaced whole. The configuration model spec lists the keys under each
rule.

## Validation boundary (decision:parallel-workspaces/workspace-configuration/validation-boundary)

**Decider:** product owner and engineering

Unknown keys are ignored without a message, except in the closed sets the configuration model spec names. `[provision]`
and `[[service]]` are validated only when a command needs them.

## Concurrent edits and recovery (decision:parallel-workspaces/workspace-configuration/write-conflicts)

**Decider:** engineering and product owner

Configuration edits preserve comments, order, and unrelated content, and no conflict is reported, as in the reference
tool. Writes are atomic by rename and serialised by an advisory lock on the `.winter/` directory, which creates no file.
