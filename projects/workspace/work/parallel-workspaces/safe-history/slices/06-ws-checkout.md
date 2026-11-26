---
name: ws-checkout
depends-on:
  - safety-probes
  - epic:parallel-workspaces/environment-lifecycle
  - epic:parallel-workspaces/coordinated-git
---

# The `ws checkout` command

Adds `CheckoutResult`, `RepoCheckout`, `Previous`, `CheckoutReport`, `checkout`, and the
`ws checkout ENV FEATURE_BRANCH` command of [checkout.md](../spec/checkout.md): ENV checked by `env_name::check` of
`epic:parallel-workspaces/environment-lifecycle` and selected as `ENV/*` through `targets::select` with
`EnvRule::Named`, the refs expanded by `ref_tokens::expand` of `epic:parallel-workspaces/coordinated-git`, an
all-or-nothing preflight over the probes of `safety-probes` and the untracked-blocker check, then per worktree the env
branch moved with `git checkout -B`, its tracking written, and `set_push_default_upstream` of
[push-default.md](../../environment-lifecycle/spec/push-default.md). No fetch runs and nothing prompts. Its row is the
`ws checkout` row of
[command-surface.md](../../../../delivered/parallel-workspaces/component-runtime/spec/command-surface.md), and its
document shape A. Delivers `product:adopt-feature-history`, the checkout bullets of `product:destructive-target-review`
and `product:history-operation-recovery`, and `engineering:destructive-fixture-confinement` for checkout, under
`decision:parallel-workspaces/safe-history/adoption-policy`,
`decision:parallel-workspaces/safe-history/pointer-recovery`, and `decision:parallel-workspaces/partial-execution`,
whose `previous`, `refused-untracked`, `failed`, and `unattempted` are registered deviations.

## Change

- `crates/core/src/history/checkout.rs` holds the types and `checkout`, which takes the start path: root discovery and
  the load, a failure `CheckoutError::Config`, then the drift warning, the name check, selection, expansion, then the
  preflight of checkout.md in its order: unknown branch unless `--new`, missing ref, and unless `--force` dirt, HEAD
  abandonment, env-branch abandonment, and untracked paths in the target's way, each target taking its first refusal;
  any refusal is the report with `aborted` true and nothing written. A failure before any effect is `CheckoutError`,
  never a `Termination`.
- Execution, serially in target order: `previous` captured, `git checkout [--force] -B <ENV> --no-track <target_rev>`,
  `branch.<ENV>.remote` and `branch.<ENV>.merge` written directly, then `push.default=upstream`; a failure is `failed`,
  with `previous` once the pointer moved, and every later target `unattempted` and untouched.
- `crates/core/src/commands/ws/checkout.rs` registers the command with `--force`, `--new`, and `--json`, makes its one
  call to `checkout`, maps a `CheckoutError` to its `Termination` as checkout.md, Domain, gives, writes the document
  once at the end or the text table with its aborted, complete, or stopped lines and hint, and exits 1 when any repo is
  refused, failed, or unattempted.
- `crates/core/topics/ws/checkout.md`, referenced from the `ws` group topic: checkout acts on every non-pinned worktree
  of one environment, adopts and connects in one step, reads local refs only, and leaves pointer moves alone to
  `ws reset`.

## Acceptance

- Component tests through `InProcess` over a workspace root and fixture remote the test creates, the remote branches
  fetched by the test: `ws checkout alpha feat/x` moves each non-pinned worktree's `alpha` branch to `origin/feat/x`,
  attached, with `branch.alpha.remote=origin`, `branch.alpha.merge=refs/heads/feat/x`, and `push.default=upstream`, each
  `reset-feature` with `previous`; a pinned worktree is neither moved nor connected; no `git fetch` runs, as a recording
  program port shows.
- A component test: a repository without `origin/feat/x` is `reset-main` at `origin/<main>`, tracking `feat/x`.
- Component tests: a branch no repository knows makes every target `refused-unknown-branch`; with `--new` each moves to
  its main tracking the branch; a repository missing both refs is `refused-missing-ref`, also under `--force`.
- Component tests: tracked dirt is `refused-dirty`, unpushed HEAD commits `refused-abandonment`, and unpushed commits on
  `alpha` while HEAD is detached or on another branch `refused-abandonment`; each aborts the whole command with every
  worktree's branch, commits, and tracking unchanged; `--force` adopts.
- Component tests: an untracked file not in the target's way does not refuse; one at, above, or below an incoming path
  is `refused-untracked` with that path in `blocking`, the file stays, and nothing moves; `--force` overwrites it.
- A component test: with no `origin/<main>` for the abandonment probe, the command exits 1 with its `RepoError` before
  any worktree moves.
- A component test: `ws checkout alpha '{main}'` connects each env branch to its own main, two projects with different
  main branches included.
- Component tests: with `config.lock` held in the second of three worktrees, the first is `reset-feature`, the second
  `failed` with `previous` and on the new commit, the third `unattempted` and unchanged, and the command exits 1; a
  `git checkout` failure is `failed` with no `previous` and its tracking unchanged.
- Component tests: `ws checkout projects feat/x` exits 1 with `EnvName` and an ENV neither registered nor present with
  `UnknownEnv`, each before any git call; an absent worktree is warned and named in `missing`.
- A component test: over a workspace with a drifted project, the drift notice of
  [drift-warning.md](../../../../delivered/parallel-workspaces/repository-catalog/spec/drift-warning.md) is written on
  stderr before the result, also under `--json`, through `drift::detect` and `drift::notice_line`.
- `cargo dev gate slice` exits 0.
