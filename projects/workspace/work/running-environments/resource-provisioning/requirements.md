# Resource provisioning requirements

For `epic:running-environments/resource-provisioning`.

## Run ordered preparation (product:ordered-preparation)

An engineer can repeat declared dependency, resource, and data preparation for one or several environments without
performing repository setup by hand.

Acceptance:

- Full preparation runs dependency, resource, then data; an explicit stage runs only that stage, and resource-plus-data
  preparation is available without treating another environment operand as a stage.
- Workspace, environment, and per-worktree scopes have distinct, disclosed working directories; an optional declared
  project narrows an environment handler to that worktree.
- Within a stage, workspace precedes environment then worktree scope, with workspace-authored before extension-authored
  handlers and stable declaration order within those groups.
- Workspace and extension declarations contribute under the same validation rules; missing, invalid, or skipped
  declarations cannot disappear from a result presented as complete.
- Repeated apply reaches the author's declared baseline; the user is told that arbitrary handlers, not an automatic
  rollback or a resource ledger, provide idempotency.

## Target declared actions (product:targeted-resource-actions)

A person can act on one named entry or a stage without accidentally applying a broader environment operation.

Acceptance:

- Apply, destroy, reset, and artifact clean remain distinct; reset uses an explicit reset or destroy-then-apply, and an
  unavailable destroy or degraded reset is reported rather than silently implied to have happened.
- A named scope-qualified selector resolves unambiguously across sources and stages; malformed, missing, ambiguous, or
  stage-conflicting selectors fail before executing any selected handler.
- Multi-environment selection is deduplicated and deterministic; missing literal targets and unmatched globs are
  distinguishable under the agreed command result policy.
- Invalid action combinations and undeclared project targets are refused before affected execution.

## Explain preparation failures (product:preparation-partial-results)

An agent can recover from failed preparation knowing what changed and which steps were not attempted.

Acceptance:

- Commands in a handler's sequence run in order and stop at its first failure; the command-list shell-state behavior is
  declared rather than assumed to be one persistent shell.
- Ordinary command failure in one worktree does not hide the outcomes of other worktree executions for that handler.
- Failed apply stops later handlers and stages; failed destroy or reset stops at the failed handler, and a failed
  destroy leg prevents the composed apply leg.
- Artifact clean continues across failed handlers and stages; per-handler missing-worktree errors remain failures rather
  than breaking best-effort cleanup. Whole-run precondition failures are distinguished from handler failures.
- Failed environments do not suppress independent selected environments without a reported reason; launch failure and
  cancellation identify completed, failed, and unattempted targets and yield an unsuccessful aggregate result.
- Structured progress, handler results, and final outcomes stay separate from handler output and diagnostics, including
  a closing outcome for runs that stop early where output can still be written.

## Preview and confine cleanup (product:declared-cleanup-scope)

A person can remove disposable artifacts or provisioned resources without mistaking an environment target for permission
to destroy shared state. Shared-scope policy is owned by
`decision:running-environments/resource-provisioning/shared-cleanup-policy`.

Acceptance:

- Preview executes neither handler commands nor service startup, and shows resolved action, target, source, working
  directory, and required-service effects; unavailable actions and incomplete validation are visible.
- Artifact clean runs only declared clean actions, preserves handlers with none, and warns for an explicitly named entry
  that cannot clean; it is not git cleaning or an implicit dependency reinstall.
- Shared workspace effects are explicitly exposed before execution, including whether several selected environments
  repeat the same action; an environment name alone does not conceal a cross-environment blast radius.
- Approval or non-interactive authorization follows the chosen destructive policy; preview does not claim trusted shell
  commands are sandboxed or cannot reach paths outside their working directory.

## Check required services (product:required-service-preconditions)

When a handler declares runtime prerequisites, the engineer can have Workspace check and start them rather than racing
setup against stopped infrastructure. This requirement depends on `epic:running-environments/service-orchestration`, not
the other preparation requirements.

Acceptance:

- Dependency installation does not require a service provider; declared resource/data prerequisites are checked only for
  handlers actually selected to act, with an explicit bypass identified in the outcome.
- References to another feature environment, malformed requirements, unknown services, and unavailable providers are
  diagnosed; catalog validation does not masquerade as runtime readiness.
- The result distinguishes already satisfied requirements, services started, failed checks, and bypasses; startup reach
  and whether started services remain running are disclosed.
- State-versus-health policy and post-start verification are chosen under
  `decision:running-environments/resource-provisioning/service-precondition-policy`; missing observation is not silently
  treated as proof that the handler can safely run.

## Reach a working environment from one command (product:composed-environment-bringup)

An engineer can request the charter's fully working environment in one safe-to-repeat operation: repositories on their
requested branches, resources provisioned, and services brought up. This requirement integrates
`epic:parallel-workspaces/environment-lifecycle` and `epic:running-environments/service-orchestration` without making
base stage execution or registry identity depend on completion of those integrations.

Acceptance:

- The composed operation orders structural reconciliation of repositories/branches, then provisioning, then workload
  service startup; it reports each phase and the scope it affects.
- Structural failure prevents provisioning and service startup for that environment; provisioning failure prevents its
  workload service startup, even if some repositories or resources were already changed.
- Infrastructure explicitly required during provisioning follows `product:required-service-preconditions`; such startup
  is disclosed as a preparation prerequisite, not permission to launch the workload after failed provisioning.
- Safe repetition converges already completed work without discarding unrelated changes; refusal or interruption states
  retained structural/resource effects and phases not attempted, including independent environment outcomes.
- Success establishes the requested working state under the agreed readiness rule, not merely completion of three
  command invocations. Readiness proof, persistence, and invalidation remain under
  `decision:running-environments/service-orchestration/readiness-enforcement`.
- The promised composition is not described as existing enforcement: the legacy provision-before-run rule is a workflow
  requirement and does not supply a readiness ledger.

## Coordinate resource teardown (product:resource-teardown-integration)

An engineer destroying an environment can clean its declared data and resources while its files still exist, or
deliberately omit that cleanup when recovering a broken environment.

Acceptance:

- Integrated teardown attempts data destroy then resource destroy before provider shutdown and structural removal;
  dependency destruction is not implied.
- The operator can preview or explicitly skip resource teardown without starting services merely to stop them.
- Strict failure preserves the environment for recovery; a continue-on-error mode still reports failed resource cleanup
  even when files are removed, and exposes unverified provider cleanup rather than calling it confirmed success.
- Shared-scope reach follows `product:declared-cleanup-scope`; lifecycle integration does not claim a persisted
  successful-provision gate already exists.
