# Workspace diagnostics requirements

For `epic:agent-ready-operation/workspace-diagnostics`.

## Locate workspace health problems (product:workspace-health-inventory)

An engineer can distinguish a workspace materialization problem from a content convention violation and identify the
affected repository, environment, declaration, or harness projection.

Acceptance:

- Checks cover required native tools, readable configuration, declared repository presence, worktree presence and
  expected local branches, environment/port registry consistency, and inert named environment-variable bands.
- When the corresponding capabilities are available, checks also cover extension declarations, generated entry-point and
  skill/agent projections, agent-model resolution and collisions, provision declarations, and service-provider binding
  and compatibility.
- Each observation identifies its subject and source; absent optional capabilities are distinguishable from broken
  configured ones, and disabled contributions are not reported as checks that passed.
- Prerequisites describe the replacement product; Python is not reported as required for first-party Workspace behavior.

## Inspect without silently repairing (product:diagnostic-repair-separation)

An engineer can inspect built-in health findings before deciding whether to reconcile or repair the workspace.

Acceptance:

- Built-in checks do not rewrite configuration or projections, create worktrees, allocate resources, install tools,
  provision data, or start services.
- A finding can offer a repair hint, but displaying it does not run that repair.
- The guidance distinguishes built-in read-only checks from configured external contributions that execute trusted code.

## Add trusted health checks (product:contributed-health-observations)

A workspace or extension owner can add project-specific checks without replacing the built-in health coverage.

Acceptance:

- Contributions identify their owner and run only under the workspace's adopted contribution policy.
- A missing, non-executable, directory-escaping, unresponsive, or failing contribution is identified without hiding
  observations from other checks.
- Invalid contribution output is visible as a contract problem rather than a healthy result; valid observations are
  retained when the same contribution later fails.
- Contribution execution is described as trusted external program execution, not as a sandbox or a no-write guarantee.

## Judge health without reading a terminal (product:actionable-health-outcomes)

An engineer or agent can distinguish passes, warnings, failures, and incomplete evidence and act on the result.

Acceptance:

- Human and non-interactive machine consumption identify each observation's severity, message, source, and available
  remediation, with a truthful overall result.
- Warnings alone do not make the health run unsuccessful; a failed check does.
- A contribution's unsuccessful execution cannot be hidden by earlier passing observations or empty output.
- Machine-readable results remain consumable without removing human progress or diagnostics.

## Explain failure before checks can run (product:diagnostic-startup-failure)

An engineer receives actionable information when configuration or workspace discovery prevents a health run from
starting; `decision:agent-ready-operation/workspace-diagnostics/degraded-startup` owns whether partial checks are
offered.

Acceptance:

- A startup failure identifies the unusable configuration or missing workspace condition without an ordinary traceback.
- A failed startup never reports that zero checks passing means the workspace is healthy.
- Any checks omitted because their prerequisites were unavailable are distinguishable from completed checks.
