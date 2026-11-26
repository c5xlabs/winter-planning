# Parallel workspace requirements

For `initiative:parallel-workspaces`. These bind every epic in the initiative.

## Name refs relative to each repository (product:ref-tokens)

An engineer can name each repository's own main branch in one command across repositories whose main branches differ.

Acceptance:

- `{main}`, `{master}`, and `{default}` anywhere in a ref, except directly after `@`, expand to the repository's
  configured main branch.
- An unknown token, or a repository with no main branch configured, refuses the command before any Git call.
- Results show the ref as the engineer typed it.

## Push where each worktree tracks (product:upstream-push-default)

An engineer can run a plain `git push` from a checkout Workspace connected and reach the branch that checkout tracks.

Acceptance:

- Connecting or adopting a worktree sets `push.default=upstream` in its project's shared Git configuration, and so does
  creating one that is pinned or inherits a sibling's upstream.
