# Mission

Winter Workspace lets many coding agents work in parallel on a system that spans many repositories, each in its own
isolated, fully running feature environment, with no collisions. It serves engineers whose work crosses repositories and
who want their agents, and themselves, to stop doing the setup a multi-repository system demands by hand.

## Scope filter

A proposed feature belongs when it removes work a person or an agent would otherwise repeat across repositories or
environments — git across many checkouts, an environment's ports and data, its running services, the context an agent
needs — or when it keeps one feature environment from disturbing another.

## Success criteria

1. **Parallel without collisions.** `persona:application-architect` runs several agents at once, each in its own feature
   environment, and none of them ever sees another's worktrees, ports, data, or services.
2. **A working environment from one command.** For `persona:application-architect`, a new feature environment reaches a
   running state — every repository on its branch, its resources provisioned, its services up — from one command that is
   safe to run again.
3. **One command across every repository.** `persona:application-architect` fetches, pulls, pushes, and inspects changes
   across every repository of an environment at once, never one checkout at a time.
4. **Any harness.** `persona:harness-engineer` brings the coding harness they prefer, and the same skills, agents, and
   context reach it.
5. **Agent-ready answers.** Every command answers with a result an agent can parse or a refusal that says why, so
   `persona:senior-se` can trust an agent to drive it.

## Non-goals

- **A fleet.** Running agents unattended and routing their work belongs to Winter Fleet; a workspace is useful with one
  person and one agent.
- **A service runtime of its own.** Services run under the provider a person already uses; the workspace orchestrates
  them and never becomes one.
- **A fork of the tool.** A workspace is its own configuration and context, never a copy of the suite's source.
