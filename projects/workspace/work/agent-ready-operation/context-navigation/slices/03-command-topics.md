---
name: command-topics
depends-on:
  - topic-map
code:
  - winter-workspace/crates/core/src/registry.rs::REGISTRY
---

# Every visible command and group has a topic

Adds the coverage test over the command registry and the topics the build's commands and groups lack. Builds
[spec.md](../spec.md), Coverage of commands. Delivers `product:framework-guidance` and the branch-and-index bullet of
`product:progressive-context`, under `decision:agent-ready-operation/context-navigation/setup-guidance` for what it
leaves to the setup topics.

## Change

- A unit test in `crates/core/src/topics.rs` checks spec.md, Coverage of commands, over `REGISTRY` and `TOPICS`: each
  visible command has a topic whose id is its path joined by `/`, each group holding one has a topic named for the
  group, and an alias needs none.
- `crates/core/topics/` gains a topic for each command and group the build holds when this lands, among them `ws`,
  `ws/index`, `repo`, and `repo/remove`. Each describes only that build's commands and options; each group topic
  references its commands, and `root` references each group and each command outside a group.
- `context/architecture/changes.md`, A command, states that a change adding a command adds its topic under
  `crates/core/topics/`, and a change adding a group adds the group's topic.

## Acceptance

- The coverage unit test passes on the tree, and fails naming the path when a visible command's topic is removed, when a
  group's topic is removed, and when a command is added without one; a hidden command and an alias need none.
- The build accepts the added topics, so each reference among them resolves and each is reachable from `root`, by the
  build rules of `topic-map`.
- `cargo dev gate slice` exits 0.

## Out of scope

- The `context` command's own topic: `context-command`.
- The `setup` topics: `epic:parallel-workspaces/workspace-bootstrap` and
  `epic:running-environments/service-orchestration`.
- That a topic describes only the build's commands: held in review under `product:framework-guidance`, not by a test.
