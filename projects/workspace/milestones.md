# Milestones

Where Winter Workspace is headed, as destinations in the work of the people it serves, in the order they are meant to be
reached. Each milestone names the epics needed to reach it; each epic's own status is the truth, so this is a plan, not
a status board. Once a milestone is reached, it moves to the [delivered ledger](./delivered.md).

The milestones are useful destinations across the four initiatives. Intermediate milestones name their required epics;
the final rough-parity milestone needs all four initiatives, rather than owning them as one large initiative.

## Declare a workspace without forking Winter (milestone:declare-workspace)

An engineer can create and inspect a workspace, declare its repositories and local settings, and give an agent guidance
that matches the selected tool. They do not clone Winter's framework source to start configuring their own system.

| Epic                                               | Status  |
| -------------------------------------------------- | ------- |
| `epic:parallel-workspaces/component-runtime`       | planned |
| `epic:parallel-workspaces/workspace-configuration` | planned |
| `epic:parallel-workspaces/repository-catalog`      | planned |
| `epic:parallel-workspaces/reproducible-inputs`     | planned |
| `epic:agent-ready-operation/context-navigation`    | planned |
| `epic:parallel-workspaces/workspace-bootstrap`     | planned |

## Work across isolated branches (milestone:parallel-branches)

An engineer can put two agents in separate feature environments, move and inspect work across repositories, and include
nested workspaces deliberately. Branch operations protect local work and explain any partial completion.

| Epic                                             | Status  |
| ------------------------------------------------ | ------- |
| `epic:parallel-workspaces/environment-lifecycle` | planned |
| `epic:parallel-workspaces/coordinated-git`       | planned |
| `epic:parallel-workspaces/safe-history`          | planned |
| `epic:parallel-workspaces/stacked-history`       | planned |
| `epic:parallel-workspaces/nested-workspaces`     | planned |
| `epic:running-environments/scope-environment`    | planned |

## Run the system in each environment (milestone:running-environments)

An engineer can provision a feature environment and run its declared services with tmux, Docker Compose, or
process-compose while another environment remains running. Failures identify the affected scope and preserve unrelated
resources.

| Epic                                                 | Status  |
| ---------------------------------------------------- | ------- |
| `epic:parallel-workspaces/environment-lifecycle`     | planned |
| `epic:running-environments/scope-environment`        | planned |
| `epic:running-environments/resource-provisioning`    | planned |
| `epic:running-environments/service-orchestration`    | planned |
| `epic:running-environments/provider-tmux`            | planned |
| `epic:running-environments/provider-docker`          | planned |
| `epic:running-environments/provider-process-compose` | planned |
| `epic:agent-ready-operation/executable-extensions`   | planned |

## Bring the team's harnesses and extensions (milestone:agent-ready-workspace)

An engineer installs the team's context and executable extensions, chooses the harness and models they use, and gives
agents the same managed skills and instructions without overwriting personal work.

| Epic                                                | Status  |
| --------------------------------------------------- | ------- |
| `epic:agent-ready-operation/context-navigation`     | planned |
| `epic:agent-ready-operation/executable-extensions`  | planned |
| `epic:agent-ready-operation/extension-management`   | planned |
| `epic:agent-ready-operation/agent-model-resolution` | planned |
| `epic:agent-ready-operation/harness-projection`     | planned |

## Understand the workspace before acting (milestone:operate-with-confidence)

Engineers and agents can diagnose their workspace, inspect context dependencies, find artifacts, and use a terminal
dashboard without acquiring a second set of safety rules or requiring the dashboard for automation.

| Epic                                               | Status  |
| -------------------------------------------------- | ------- |
| `epic:agent-ready-operation/dependency-graph`      | planned |
| `epic:agent-ready-operation/workspace-diagnostics` | planned |
| `epic:agent-ready-operation/context-quality`       | planned |
| `epic:agent-ready-operation/workspace-dashboard`   | planned |
| `epic:agent-ready-operation/artifact-space`        | planned |

## Replace the Python Workspace tool (milestone:rust-workspace-parity)

An existing user can install and adopt the Rust Workspace component with the inventoried capabilities accounted for,
deliberate differences documented, and no supported first-party Python runtime path. Unresolved redesigns remain
blockers rather than being treated as implicit permission to drop functionality.

| Epic                               | Status  |
| ---------------------------------- | ------- |
| `initiative:parallel-workspaces`   | planned |
| `initiative:running-environments`  | planned |
| `initiative:agent-ready-operation` | planned |
| `initiative:rust-adoption`         | planned |
