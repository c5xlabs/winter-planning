# Workspace configuration decisions

## Element type validation (decision:parallel-workspaces/workspace-configuration/element-type-validation)

**Decider:** product owner

A value that is not an array of strings for `git_excludes`, or for a repository entry's `git_excludes` or `cmd`, and a
non-string `main_branch` or `ref`, is a configuration error naming the key, exit 1; list keys with a coercion rule of
their own, such as `env_aliases` and `capabilities.<slot>`, keep that rule. The reference tool reads a bare string for
`git_excludes` or `cmd` as one entry per character and a table as its keys, and fails unhandled only on a non-iterable
value or a non-string element, so refusing all of them is a registered deviation.

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
