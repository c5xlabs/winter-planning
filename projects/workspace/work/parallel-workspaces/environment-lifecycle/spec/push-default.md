# Push default

The `push.default=upstream` write that lets a plain `git push` in a managed checkout reach the branch it tracks. The git
calls go through the program port of
[child-processes.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/child-processes.md). Callers: the
worktree reconciliation of `ws init` here ([init.md](./init.md)), `ws connect` of
`epic:parallel-workspaces/coordinated-git` ([tracking.md](../../coordinated-git/spec/tracking.md)), and `ws checkout` of
`epic:parallel-workspaces/safe-history`; each links this leaf. Reference locations are paths under
`tools/winter-cli/src/winter_cli/` of `winter-workspace` at `127cdf2`.

## Domain

```rust
// crates/core/src/git.rs — product:parallel-workspaces/upstream-push-default
/// The effective push.default the checkout at `path` reads, every scope merged; None when unset.
pub(crate) fn push_default(programs: &dyn RunPrograms, tracer: &dyn CommandTracer, repo: &str, path: &Path)
    -> Result<Option<String>, GitFailure>;
/// Writes push.default=upstream to the repository configuration of the checkout at `path`.
pub(crate) fn set_push_default_upstream(programs: &dyn RunPrograms, tracer: &dyn CommandTracer, repo: &str,
    path: &Path) -> Result<(), GitFailure>;
// both are GitOperation::Config, span "git config"
```

## Interfaces

```text
# reference: modules/workspace/internal/write_repo_repository.py:601-604,
#            modules/workspace/internal/gitpython_repository.py:190-206
push_default(path)               git config --get push.default
                                   exit 0 -> the value, less its final newline; exit 1 -> None; other ->
                                   GitFailure::Repo { Config, repo }
set_push_default_upstream(path)  git config --local push.default upstream
                                   --local writes $GIT_COMMON_DIR/config, the file the source checkout and every
                                   worktree of the project share, also when extensions.worktreeConfig is on: the
                                   value is the project's, not the worktree's, as GitPython's config_writer()
                                   writes it
                                   non-zero exit -> GitFailure::Repo { Config, repo }
```

```text
# when each caller writes it; reference: modules/workspace/env_checkout_service.py:59-61,210-219,
#                                        modules/workspace/init_service.py:695-740
ws connect      every connected worktree, after its branch.<head>.* writes, unconditionally
ws checkout     every adopted worktree, after its tracking writes, unconditionally
ws init         pinned worktree: when push_default is not "upstream", write it and name "push.default=upstream" in
                the pinned_tracking_set action; non-pinned: written with each inferred upstream it sets
a write that fails ends that worktree's step with its RepoError, as the caller's leaf handles it
```
