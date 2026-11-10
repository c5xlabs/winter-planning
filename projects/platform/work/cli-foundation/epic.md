---
epic: cli-foundation
refinement: outline
status: planned
---

# Winter CLI foundation

A standalone epic; its product requirements live in [requirements.md](./requirements.md).

## The problem

A person approaching Winter needs to know what the platform command does and which version they are running before
setting up a workspace or a fleet. Without a command that answers those questions on its own, their first interaction
depends on a product installation and setup they have not chosen yet.

Maintainers and coding agents also need a foundation they can extend without choosing the project's architecture again
for each change. When command structure, dependency boundaries, and verification conventions are left implicit, even a
small addition turns into design work and reviewers must reconcile incompatible approaches instead of judging behavior.
People trying the tool also need a published version they can install without becoming contributors or setting up a Rust
toolchain.

## The outcome

The person can run the platform executable, discover its supported options, and identify its version from any directory.
This first executable has a small, truthful surface: help and version, with an explanation when an invocation is not
supported. It stands on its own rather than wrapping the existing Workspace CLI
(`decision:cli-foundation/standalone-command`).

Maintainers have a repeatable development loop with automated feedback. Agents can extend the system within a documented
architecture without making excessive project-level architectural decisions: routine work follows established patterns,
and a change that needs a new architectural decision has a clear boundary at which to ask.

The foundation includes a dedicated test project, enforced quality and architectural checks, and repository-owned
context that gives code and architecture reviews the same rules contributors use. Maintainers can publish a versioned
release, and users can install its prebuilt executable through Homebrew without building from source.

In development, a working Rust sample component is launched as a separate executable through the platform CLI. It
demonstrates the module boundary with observable argument, output, and exit behavior, rather than leaving modularity
as a diagram; the sample is absent from released installations (`decision:cli-foundation/development-sample`).

## What this epic is not

- Workspace, hub, or runner commands, installed-product discovery, or general product dispatch. The only dispatched
  component in this epic is the explicitly selected development sample.
- A bridge to the existing Python Workspace CLI, command aliases for it, or a migration compatibility layer.
- Workspace discovery, configuration loading, version-pin resolution, or development-source overrides.
- Product-component installation, downloads, or self-update. Publishing and Homebrew installation cover the platform
  executable only; development and verification do not replace the user's installed command.
