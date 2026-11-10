# Mission

Build an autonomous software development system that enables engineers to deliver exceptionally high-quality results
across the widest possible range of applications. Engineers work primarily as humans on the loop: they set direction and
monitor outcomes, intervening when their judgment is needed.

## Scope filter

A proposed product belongs in the suite when it provides reusable building blocks that engineers can assemble and extend
to get trustworthy, landed software from coding agents. It must be useful on its own: a person can take it without
taking any other product in the suite. Capabilities belong when they directly support that development process; adjacent
infrastructure belongs behind an integration with a dedicated tool or provider.

## Success criteria

1. **Each product stands alone.** `persona:harness-engineer` can adopt Workspace without Fleet, or Fleet without
   Workspace, and get full value from either; taking both adds to each rather than unlocking it.
2. **Agents get a straight answer.** Every command an agent runs on behalf of `persona:harness-engineer` gives a result
   it can parse or a reason it can act on, never a silent partial success.
3. **Trust is earned, not demanded.** `persona:senior-se` can start with a human check at every step and remove checks
   as confidence grows, in every product of the suite.
4. **Governance at scale.** `persona:application-architect` defines quality objectives for the application and governs
   their pursuit across many concurrent agent workflows. Evidence from throughout the development lifecycle lets them
   assess and drive improvement in every quality of interest without supervising each agent action.

## Non-goals

- **A coding harness.** Agents come from the harness a person already uses; the suite runs and coordinates them and
  never reimplements one.
- **A general-purpose observability platform.** Winter exposes the evidence needed to understand and improve agent
  workflows; broader telemetry needs are served by integrations with dedicated observability tools.
- **A cloud agent provider.** Winter coordinates agents in execution environments the engineer chooses. Hosting agents
  as a managed cloud service is outside the suite's scope.
- **Tied to one tool.** No product is welded to one harness, forge, or tracker; each is a binding that can be swapped.
- **A bundle.** No install forces a person to take a product they didn't ask for.
