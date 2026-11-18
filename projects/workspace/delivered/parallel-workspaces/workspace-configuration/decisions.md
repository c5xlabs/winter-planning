# Workspace configuration decisions

## Element type validation (decision:parallel-workspaces/workspace-configuration/element-type-validation)

**Decider:** product owner

A value that is not an array of strings for `git_excludes`, or for a repository entry's `git_excludes` or `cmd`, and a
non-string `main_branch` or `ref`, is a configuration error naming the key, exit 1; list keys with a coercion rule of
their own, such as `env_aliases` and `capabilities.<slot>`, keep that rule. The reference tool reads a bare string for
`git_excludes` or `cmd` as one entry per character and a table as its keys, and fails unhandled only on a non-iterable
value or a non-string element, so refusing all of them is a registered deviation.

## Refusal order within a step (decision:parallel-workspaces/workspace-configuration/within-step-order)

**Decider:** product owner

Parity covers which numbered Load step refuses a configuration, and that refusal's error kind. When one step meets
several faults, which of them the error names follows the slices' stated order and may differ from the reference tool's;
every such refusal exits 1 with the tree unchanged, as in the reference.

## Pair keys of a shared array (decision:parallel-workspaces/workspace-configuration/pair-keys)

**Decider:** product owner

When the merge reads a shared array of pairs as a table, a pair whose first item is not a string becomes a string key in
the binary, its text. The reference tool keeps it as a non-string key, so the two tools group the pairs differently: the
reference folds keys Python holds equal, such as `1` and `true`, into one, which the binary keeps apart; the binary
folds a non-string key into a string key of the same text, such as `1` and `"1"`, which the reference keeps apart, and
then checks only the later value. Either way the checks that read the table can accept or refuse differently from the
reference. This is a registered deviation.

## A configuration path that is a directory (decision:parallel-workspaces/workspace-configuration/directory-config-path)

**Decider:** product owner

A `.winter/config.toml` or `.winter/config.local.toml` that exists and is not a regular file is `ConfigError::FileRead`
with the `Io` cause, exit 1. The reference tool reads such a path as missing; refusing it is a registered deviation.

## State file compatibility (decision:parallel-workspaces/workspace-configuration/state-file-compatibility)

**Decider:** engineering

`.winter/state.toml` and `.winter/config.lock` are read and written in the reference tool's formats. A `state.toml` that
fails to parse is read as empty, with no message, and replaced on the next write.

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
