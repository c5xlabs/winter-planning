# Workspace dashboard requirements

For `epic:agent-ready-operation/workspace-dashboard`.

## Compare workspace state (product:dashboard-workspace-overview)

An engineer can compare project worktrees and standalone repositories without losing the distinction between an
environment, a repository, and its source checkout.

Acceptance:

- The overview identifies environments, worktrees, standalone repositories, pinned repositories, dirtiness, and
  divergence from main and from the configured upstream where available.
- No upstream, a configured upstream that has not appeared remotely, and an existing upstream are distinguishable.
- Source-checkout dirtiness/divergence and per-worktree remote differences remain visible; an environment with several
  distinct remote branches is not presented as uniformly connected to just one.
- Available environment service observations and extension badges are distinguishable from native repository status;
  missing or failed observations are not rendered as healthy stopped or clean state.

## Inspect the selected repository (product:dashboard-repository-detail)

An engineer can open the repository under the cursor and inspect the evidence behind its overview row.

Acceptance:

- Opening a worktree retains the chosen repository within its environment; opening a standalone shows that repository.
- Details show branch, upstream condition, divergence, changed paths, and navigable commit topology where present.
- Moving among repositories updates the detail; removed or unavailable selections have an explicit fallback or error
  rather than silently displaying another repository's old details.
- Contributed detail content is distinguishable from built-in information, and one failing panel does not blank the
  remaining detail view.

## Trust refresh and freshness (product:dashboard-freshness)

An engineer can refresh manually or let the overview poll without restarting whenever repository state changes.

Acceptance:

- Repository/environment additions and removals become visible on a successful overview refresh of the effective
  workspace declarations, not only after restarting.
- A malformed configuration during a session preserves usable last-good observations, identifies the refresh failure,
  and does not imply that stale state is current; initial failure shows an error rather than an endless loading view.
- A slower older refresh cannot replace a newer accepted refresh or move details to the wrong selection.
- The dashboard explains which settings are live and which require reopening a view or restarting, as settled by
  `decision:agent-ready-operation/workspace-dashboard/freshness-boundary`.

## Navigate with a recoverable keymap (product:dashboard-keyboard-control)

An engineer can navigate and adapt the dashboard to workspace shape and personal keyboard habits.

Acceptance:

- Row-oriented, column-oriented, and list views show the same selected environment/repository relation; automatic layout
  follows workspace shape, and cycling layout preserves the logical selection where it still exists.
- A configured default can be inspected non-interactively; session-only layout changes do not rewrite that default.
- Stable named actions can be remapped using ordinary keys, modifiers, leader keys, and timed multi-key sequences.
- Invalid bindings and overlapping action conflicts are explained without discarding unrelated bindings; disjoint
  contribution areas can share a key, and an unconditional quit escape remains available despite a broken keymap.

## Inspect session failures (product:dashboard-error-history)

An engineer can inspect a dashboard failure without rerunning the operation that caused it.

Acceptance:

- A session-local view retains captured operation time, location, target, message, and available command/cwd/exit/error
  details across screen navigation, including configuration and contribution failures.
- Repeated polling failures remain inspectable without producing a fresh intrusive notification every time.
- The engineer can clear the retained history; retention limits or truncation are visible under
  `decision:agent-ready-operation/workspace-dashboard/error-retention` rather than silently losing evidence.
- The view does not promise persistence after dashboard exit or replace project-wide logging and tracing.

## Keep useful extension experiences (product:dashboard-contribution-outcomes)

Extension authors can contribute scoped observations, detail information, and deliberate actions without importing
extension UI code into Workspace; `decision:agent-ready-operation/workspace-dashboard/contribution-replacement` sets
that boundary and `decision:agent-ready-operation/workspace-dashboard/contribution-scope` owns the breadth of
replacement.

Acceptance:

- Environment/worktree badges, named detail panels, and keybound actions have identifiable owners and remain separable
  from native observations.
- Actions can identify workspace, environment, worktree, or standalone selection, including a multi-area action whose
  behavior follows the focused area; environment-wide actions can identify sibling worktrees.
- External action programs may use any language chosen by their owner and can request a deliberate terminal handoff
  without corrupting the dashboard on return.
- Existing command contributions and arbitrary custom screens are explicitly accounted for in the replacement scope;
  badges alone are not described as replacing commands, screens, panels, and actions.
- An unsupported contribution is named with migration guidance; it is not silently omitted or executed through a Python
  compatibility host.

## Inspect resolved agent settings (product:dashboard-agent-inspection)

An engineer can see the resolved agent model and effort, where those values came from, and whether generated copies
match the current declarations.

Acceptance:

- The dashboard and non-interactive agent inspection agree on defaults, overrides, effective values, and unmatched
  override targets, with an understandable source legend.
- Reopening or explicitly refreshing the agent view reads current effective configuration and generated-copy state.
- Load failures remain visible and inspectable without taking down the workspace overview.

## Run deliberate actions without duplicate work (product:dashboard-deliberate-actions)

An engineer can start a supported action, including workspace reconciliation from agent inspection, while knowing its
target and actual outcome.

Acceptance:

- Merely opening or refreshing a screen never reconciles, provisions, starts services, or runs a mutating action.
- A started action uses the selected target and current applicable safety rules; stale selections or unavailable
  prerequisites are refused rather than redirected to another target.
- A repeated action or reopening its screen cannot launch duplicate conflicting mutations; completion is reported once
  and the relevant observations can refresh afterward.
- Slow work remains visibly running rather than being reported successful or cancelled solely because a warning timer
  elapsed; output and undisclosed prompts do not take over the dashboard.
- Leaving a screen or quitting does not strand the terminal or report interrupted/unfinished work as complete; the
  cancellation or continuation policy is owned by
  `decision:agent-ready-operation/workspace-dashboard/action-concurrency`.
