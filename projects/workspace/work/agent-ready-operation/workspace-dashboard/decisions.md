# Workspace dashboard decisions

## Freshness boundary (decision:agent-ready-operation/workspace-dashboard/freshness-boundary)

**Decider:** product owner and engineering

Freshness follows the reference tool: the overview reloads configuration on every refresh and keeps the last good value
when the file is malformed. Detail views, keymaps, and contributions stay as they were at launch.

## Action concurrency and exit (decision:agent-ready-operation/workspace-dashboard/action-concurrency)

**Decider:** product owner and engineering

Mutating actions follow the reference tool: one runs at a time for the whole session, and a warning appears after 60 s
without cancelling the action. Quitting cancels running workers.

## Session error retention (decision:agent-ready-operation/workspace-dashboard/error-retention)

**Decider:** product owner and engineering

Session errors follow the reference tool: an unbounded in-memory list for the session, with a clear action. Nothing is
persisted or exported.

## Contribution breadth (decision:agent-ready-operation/workspace-dashboard/contribution-scope)

**Decider:** product owner

The binary loads no plugin: the tmux service badge is native, and every other `plugin.py`, plugin directory, and
plugin-contributed command is ignored with one notice. This is a registered deviation from the reference tool.

## Replace imported Python contributions (decision:agent-ready-operation/workspace-dashboard/contribution-replacement)

The in-process Python plugin seam is not reproduced and has no replacement in this build: the binary embeds no Python,
imports no plugin, and hosts no compatibility layer, under `decision:rust-replacement`. A new contribution contract is a
later capability, outside the parity build; `decision:agent-ready-operation/workspace-dashboard/contribution-scope`
states what the binary does with existing plugins.
