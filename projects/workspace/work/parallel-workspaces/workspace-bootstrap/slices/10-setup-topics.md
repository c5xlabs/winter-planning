---
name: setup-topics
depends-on:
  - init-command
  - ws-init-command
  - epic:agent-ready-operation/context-navigation
---

# The setup topics

Adds the `setup`, `setup/walkthrough`, `setup/contributing`, and `setup/project-setup` topics of
[guidance.md](../spec/guidance.md), under `decision:agent-ready-operation/context-navigation/setup-guidance`, restated
from the reference sources the spec names for this binary's commands. Delivers the setup route of
`product:create-unforked-workspace` and the setup topics `epic:agent-ready-operation/context-navigation` assigns to this
epic.

## Change

- `crates/core/topics/setup.md` and the three leaves under `crates/core/topics/setup/`; `root` references `setup`,
  and `init` references `setup` and `setup/walkthrough`, now that they exist.
- `init`'s text gains its "next" line naming `winter-workspace context setup` (creation.md, Result).

## Acceptance

- The topic build of `epic:agent-ready-operation/context-navigation` accepts the four sources: every reference
  resolves, each topic is reachable from `root`, and the root stays within `engineering:entry-size`.
- The coverage test passes: every `winter-workspace` command span in the four bodies parses to a command the build
  holds.
- A component test: `context setup` lists exactly the three leaves; `context setup/walkthrough` names `init`, `repo
  add`, and `ws init` in that order.
- `cargo dev gate slice` exits 0.
