# Workspace bootstrap decisions

## Clone confinement (decision:parallel-workspaces/workspace-bootstrap/clone-confinement)

**Decider:** product owner

Before cloning a standalone, `ws init` refuses it as that repository's failure when its location, symlinks resolved, is
outside the workspace root, is the root, lies under `.winter/` or `projects/`, or contains or lies inside another
declared repository's location; a checkout already present is verified, never moved. This answers the stricter
confinement `decision:parallel-workspaces/repository-catalog/location-policy` left to materialization, and is a
registered deviation: the reference tool clones wherever the path leads.

## Prune retention and approval (decision:parallel-workspaces/workspace-bootstrap/prune-retention)

**Decider:** product owner and engineering

`ws prune` removes an orphan only when nothing in it can be lost: a candidate with no `.git` directory, a `.git` file,
linked worktrees, a dirty or unreadable tree, a commit no remote-tracking branch holds, or a stash is retained, as is a
candidate that resolves outside the workspace root; the retained orphan keeps its managed exclude block, and there is no
discard path, so the engineer deletes retained material by hand. `--json` without `--force` or `--dry-run` is refused,
as `ws clean` refuses it, where the reference tool removes without asking; each of these is a registered deviation,
taken because the reference deletes standalone directories that are not clones, misses local-only history and stashes,
and drops the evidence a later prune needs.

## Reconciliation scope (decision:parallel-workspaces/workspace-bootstrap/reconciliation-scope)

**Decider:** product owner and engineering

`ws init` keeps the reference tool's scopes: bare reconciles source checkouts then standalones, a TARGET one feature
environment, and `--all` both. A location that already exists is verified before it is used: one that is not the top of
a git work tree, or whose `origin` URL differs from the declared `url`, is that repository's failure and its later steps
are skipped, where the reference reports `exists` and carries on; this registered deviation is taken because the later
steps write identity, excludes, and pins into whatever repository sits there. HEAD is not checked, so a clone of an
empty remote stays valid. A source checkout is cloned on its `main_branch`, where the reference takes the remote's
default HEAD (paul-gross/winter#242), a registered deviation. Concurrent invocations take no workspace-wide lock: file
updates are serialized per file, and a clone that races another fails as that repository's failure.

## New-workspace ownership (decision:parallel-workspaces/workspace-bootstrap/creation-ownership)

**Decider:** product owner and engineering

A new top-level `init` command creates a workspace in the caller's directory: an empty-but-pinned `.winter/config.toml`
holding `versions.workspace`, the eager safety text in a managed block of `AGENTS.md`, and a `CLAUDE.md` shim. It writes
no seed repositories, runs no `git init`, does not go on to reconcile, refuses a directory that already holds
`.winter/config.toml`, and never replaces an existing `AGENTS.md` or `CLAUDE.md`. A later `ws init` refreshes the block
only where `AGENTS.md` already holds it. Both are registered deviations: the reference tool's `init` is launcher-only
(`decision:parallel-workspaces/component-runtime/launcher-only-surface`), creates an empty file, and reconciles; the
binary owns creation because `product:create-unforked-workspace` needs it without the launcher or a framework clone.
