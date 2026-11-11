---
initiative: agent-ready-operation
refinement: outline
epics:
  - context-navigation
  - executable-extensions
  - extension-management
  - agent-model-resolution
  - harness-projection
  - dependency-graph
  - workspace-diagnostics
  - context-quality
  - workspace-dashboard
  - artifact-space
---

# Make the workspace ready for agents and operators

Engineers need their preferred harnesses and extensions to share usable guidance and trustworthy workspace observations.

## The problem

An agent needs the right conventions, tools, model settings, and target information before acting. Today these arrive
through several generated file layouts, extension mechanisms, and inspection surfaces. Stale copies, lost permission
settings, incomplete observations, or a diagnostic that silently repairs state prevent engineers from trusting what an
agent sees or does.

## The outcome

`epic:agent-ready-operation/context-navigation` supplies version-matched, progressively disclosed guidance and the
minimum eager safety hook. `epic:agent-ready-operation/executable-extensions` and
`epic:agent-ready-operation/extension-management` separate acquiring an extension from enabling or trusting its
contributions, without importing foreign code into Workspace. `epic:agent-ready-operation/agent-model-resolution`
explains effective model, effort, and permission settings; `epic:agent-ready-operation/harness-projection` delivers
supported harness artifacts without damaging canonical sources or hand-authored work.

`epic:agent-ready-operation/dependency-graph` makes declared module dependencies inspectable.
`epic:agent-ready-operation/workspace-diagnostics` and `epic:agent-ready-operation/context-quality` distinguish health,
convention findings, suppressed findings, and incomplete evidence. `epic:agent-ready-operation/workspace-dashboard` puts
the same observations and deliberate actions in a terminal view with recoverable errors and command-equivalent safety.
`epic:agent-ready-operation/artifact-space` gives tools a configurable destination without creating or claiming
ownership of its contents during lookup.

## Left for later

Repository acquisition and structural lifecycle belong to `initiative:parallel-workspaces`; resource and service
operations belong to `initiative:running-environments`. This initiative consumes their results instead of introducing a
second mutation policy. `initiative:rust-adoption` owns the supported-consumer transition and live-cutover evidence.

## Stance

Inspection is not repair, and reading declarations is not permission to execute contributions. A matching prefix does
not prove artifact ownership; a missing observation does not prove health. Unsupported permissions and contribution
types remain explicit, and a subprocess boundary is not a sandbox. Common context and artifact-path resolution can be
delivered independently of the full harness and dashboard integrations, so the first workspace milestone still has
essential safety guidance.
