# Mission

C5X Winter is a suite for building software with coding agents. It serves engineers who hand real work to agents and
expect it to land: Winter Workspace gives each agent an isolated, fully running copy of the whole system to work in, and
Winter Fleet runs many agents unattended and brings their work back for review or straight to the main branch. Both sit
behind one `winter` command.

The suite is built for agents as much as for the people directing them. A command an agent runs either works or says
exactly why it didn't, because an agent recovers from a clear refusal far better than from a half-finished success.

## Scope filter

A proposed product belongs in the suite when it helps a person get trustworthy, landed software out of coding agents,
and it is useful on its own: a person can take it without taking any other product in the suite.

## Success criteria

1. **Each product stands alone.** `persona:application-architect` can adopt Workspace without Fleet, or Fleet without
   Workspace, and get full value from either; taking both adds to each rather than unlocking it.
2. **One front door.** Whatever a person installs, they reach it through one `winter` command, one install, and one
   upgrade path, and two workspaces on one machine can run different versions without stepping on each other.
3. **Agents get a straight answer.** Every command an agent runs on behalf of `persona:harness-engineer` gives a result
   it can parse or a reason it can act on, never a silent partial success.
4. **Trust is earned, not demanded.** `persona:senior-se` can start with a human check at every step and remove checks
   as confidence grows, in every product of the suite.

## Non-goals

- **A coding harness.** Agents come from the harness a person already uses; the suite runs and coordinates them and
  never reimplements one.
- **Tied to one tool.** No product is welded to one harness, forge, or tracker; each is a binding that can be swapped.
- **A bundle.** No install forces a person to take a product they didn't ask for.
