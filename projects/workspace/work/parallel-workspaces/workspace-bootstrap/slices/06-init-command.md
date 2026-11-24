---
name: init-command
depends-on:
  - agents-md
  - module-scaffold
code:
  - winter-workspace/crates/core/tests/root.rs::init_is_no_command_and_the_root_help_does_not_list_it
  - winter-workspace/crates/core/tests/root.rs::REGISTERED
---

# `init` creates a workspace in the caller's directory

Adds the top-level `init [--json]` command and `bootstrap::create` of [creation.md](../spec/creation.md), with the
`init` topic of [guidance.md](../spec/guidance.md). Delivers `product:create-unforked-workspace` under
`decision:parallel-workspaces/workspace-bootstrap/creation-ownership`, a registered deviation from the reference tool's
launcher-only `init` (`decision:parallel-workspaces/component-runtime/launcher-only-surface`).

## Change

- `crates/core/src/bootstrap/create.rs` holds `create`: the refusal of an existing `.winter/config.toml`, the
  create-only write of `[versions] workspace = "<version>"`, then `agents_md::place` and `agents_md::shim`.
- `crates/core/src/commands/init.rs` registers the command, calls `create` with `cx.cwd` and `cx.identity.version`, and
  writes the shape A document or the text of creation.md, Result; a conflict is a warning on stderr and exit 0. Its
  "next" line names `repo add` and `ws init` alone, as guidance.md, lands, gives until the setup topic exists.
- `crates/core/tests/root.rs` drops `init_is_no_command_and_the_root_help_does_not_list_it`, and `REGISTERED` gains
  `init`.
- `crates/core/topics/init.md`, referencing no topic yet, and referenced from `root`.
- `context/verification/matrix.md` gains `workspace:manual-init`: run the built binary in an empty directory, then
  again, and read the files.

## Acceptance

- Component tests through `InProcess`: in an empty directory, `init --json` exits 0 with
  `{"config": "created", "agents_md": "created", "claude_md": "created"}` and the three files as creation.md gives them;
  the config loads with no repository and `versions.workspace` equal to the binary's version.
- A component test: with `.winter/config.toml` present, `init` exits 1 with `AlreadyInitialized` and writes nothing,
  `AGENTS.md` and `CLAUDE.md` untouched.
- Component tests: an existing `AGENTS.md` keeps every byte below the prepended block; a `CLAUDE.md` without
  `@AGENTS.md` is untouched, reported `"conflict"`, exit 0.
- A component test: no git process is started (the program stub records none).
- The coverage tests of `epic:agent-ready-operation/context-navigation` pass with `init` registered.
- `cargo dev gate slice` exits 0.
