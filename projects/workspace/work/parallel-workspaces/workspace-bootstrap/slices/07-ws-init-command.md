---
name: ws-init-command
depends-on:
  - checkout-steps
  - workspace-exclude
  - agents-md
---

# `ws init` reconciles source checkouts and standalones

Adds `bootstrap::reconcile` and the `ws init [--json]` command of [reconciliation.md](../spec/reconciliation.md),
Running `ws init`, Fan-out, and Events and text: the projects and standalone targets, the workspace exclude blocks,
`finalize`, and the `AGENTS.md` refresh, with the `ws/init` topic of [guidance.md](../spec/guidance.md). Delivers
`product:reconcile-repositories`, `product:bootstrap-recovery`, `engineering:bootstrap-effect-accounting`, and the
refresh half of `product:managed-content-ownership`.

## Change

- `crates/core/src/bootstrap.rs` holds `reconcile`, `InitEvent`, `Action`, `InitSink`, `RunEnd`, and the event text
  renderer `text`; the fan-out runs at most 4 repositories at once on `std::thread::scope` and hands events to the sink
  on the calling thread in arrival order.
- The projects target writes the `winter-workspace/workspace-skills` block after its fan-out.
- `crates/core/src/commands/ws/init.rs` registers `ws init [--json]`, builds the sink over `output::Results`, writes
  shape C events or the rows of `bootstrap::text`, and ends 1 when the run reported a failure. TARGET and `--all` are
  not registered; `epic:parallel-workspaces/environment-lifecycle` adds them.
- `crates/core/topics/ws/init.md`, referencing no topic yet, since `ws/prune` lands later, and referenced from the `ws`
  group topic; it states what bare `ws init` leaves out.
- `context/verification/matrix.md` gains `workspace:manual-ws-init`.

## Acceptance

- Component tests through `InProcess` with fixture remotes: a first run clones every project and standalone, writes
  `winter-dir/projects`, `winter-workspace/artifacts`, `winter-workspace/workspace-skills` with the workspace prefix,
  `winter-config`, and one block per standalone, and emits the events of reconciliation.md in target order; a second run
  clones nothing, writes no exclude file, and exits 0.
- A component test: a failing project does not stop another project or the standalone target; the run exits 1 and the
  failing project's `target_completed` reports `success: false`.
- A component test: removing a standalone from the configuration and running again strips its block alone, reported with
  the remaining names, or `cleared`.
- A component test: a declared project-repository extension (a `winter-ext.toml` in its checkout) keeps its path block.
- A component test: an `AGENTS.md` holding an outdated safety block is refreshed and reported `agents_md_updated`; one
  without the block is left untouched.
- A component test: a root that is not a git repository reconciles every repository and writes no exclude file.
- A component test: with the root a git repository, `git status --porcelain` there prints nothing after a run that
  cloned a project and a standalone.
- A component test: a workspace with projects alone, or under `adopt_extensions = "none"`, writes the `winter-config`
  block and emits no `winter-extensions` event.
- A component test running two `ws init` processes of the built binary at once over one workspace: both exit 0 or one
  reports a clone race as that repository's failure; a third run then exits 0, every block and the lock record are
  present once, and no clone is left without a HEAD.
- `context/verification/matrix.md`'s `workspace:manual-ws-init` includes the interrupt flow: SIGINT during a slow `cmd`
  ends the run with exit 1 and the aborted notice, and the next run reconciles the workspace.
- A unit test of the fan-out over a recording `RunPrograms`: never more than 4 repositories in flight, and one
  repository's events keep their order.
- `ws init alpha` is a usage error, exit 2.
- `cargo dev gate slice` exits 0.
