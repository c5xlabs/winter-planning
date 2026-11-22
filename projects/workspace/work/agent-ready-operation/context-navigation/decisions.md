# Context navigation decisions

## Configuration inspection (decision:agent-ready-operation/context-navigation/configuration-inspection)

**Decider:** product owner

The context command does not explain effective configuration, and no command for it is added now, because the context
covers framework guidance rather than live workspace state;
`product:parallel-workspaces/workspace-configuration/configuration-explanation` stays proposed with no owning epic.

## Safety hook delivery (decision:agent-ready-operation/context-navigation/safety-hook)

**Decider:** product owner

The safety rules reach the agent as eager text only, with no session-start hook, because eager text works on every
supported harness without per-harness configuration and the first milestone must not wait for
`epic:agent-ready-operation/harness-projection`.

## Context addressing (decision:agent-ready-operation/context-navigation/topic-addressing)

**Decider:** product owner

Topics are addressed by logical ids from a topic map compiled into the binary, not by file path, so the guidance can be
reorganized without breaking requests. The context holds only Winter's framework guidance: extensions and projects are
repositories, so their Markdown stays readable by path and needs no registration.
