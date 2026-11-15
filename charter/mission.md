# Mission

Winter is an open-source, developer-owned platform and suite of tools for people working in software to automate every
aspect of software development and make agents more capable and efficient.

An individual can bring their tools and automation to any project they work on. A team or organization can use the same
platform to govern how software is developed, setting shared expectations and verifying that the work meets them.
Individual ownership and organizational governance must work together, rather than requiring people to give up one to
gain the other.

Winter's mission is to give the software workforce access to the tools they need to learn and shape the work as it
changes: from software engineering to harness engineering, to factory engineering, and to whatever comes next. People
can build on what they know, automate more as they learn, and choose where their judgment remains essential.

## Scope filter

A proposed product belongs in the suite when it helps people automate work in the software development lifecycle,
improve agents' capabilities or efficiency, or govern and learn from that work. It must preserve developers' ability to
own, adapt, and carry their automation between projects, and be useful without requiring every other Winter product.
Adjacent infrastructure belongs behind an integration with a dedicated tool or provider rather than being rebuilt
without a software-development need.

## Success criteria

1. **Developer-owned and portable.** `persona:harness-engineer` can inspect, modify, and extend the open-source tools,
   keep their automation under their control, and bring it to another project without depending on one employer,
   proprietary service, or tool provider.
2. **Each product stands alone.** `persona:harness-engineer` can adopt Workspace without Fleet, or Fleet without
   Workspace, and get full value from either; taking both adds to each rather than unlocking it.
3. **Agents get a straight answer.** Every command an agent runs on behalf of `persona:harness-engineer` gives a result
   it can parse or a reason it can act on, never a silent partial success.
4. **Trust is earned, not demanded.** `persona:senior-se` can start with a human check at every step and remove checks
   as confidence grows, in every product of the suite.
5. **Governance at scale.** `persona:application-architect` defines a team's or organization's development policies and
   quality objectives and governs their pursuit across many concurrent agent workflows. Evidence from throughout the
   development lifecycle lets them assess and drive improvement without supervising each agent action.
6. **A path to the next kind of work.** `persona:senior-se` can learn to build and tune harnesses using the tools they
   already use for development. `persona:harness-engineer` can carry those skills and automation into operating and
   improving software factories, without replacing their whole toolset at each transition.
7. **Improvement backed by evidence.** `persona:harness-engineer` can compare outcomes, quality, time, and resource use
   across runs to tell whether a change makes agents more capable or efficient.

## Non-goals

- **A coding harness.** Agents come from the harness a person already uses; the suite runs and coordinates them and
  never reimplements one.
- **A general-purpose observability platform.** Winter exposes the evidence needed to understand and improve agent
  workflows; broader telemetry needs are served by integrations with dedicated observability tools.
- **A cloud agent provider.** Winter coordinates agents in execution environments the engineer chooses. Hosting agents
  as a managed cloud service is outside the suite's scope.
- **Tied to one tool.** No product is welded to one harness, forge, or tracker; each is a binding that can be swapped.
- **A bundle.** No install forces a person to take a product they didn't ask for.
- **A prescribed career or autonomy level.** Winter supports people as their work changes; it does not require a job
  title, a fixed progression, or surrendering human judgment to use its tools.
