# Suite product requirements

## Dogfood governed context (product:governed-context)

Every project in the suite dogfoods its own agent-facing context structure under shared engineering governance and
`winter-canon`, so contributors and reviewers use the same context conventions the suite supports
(`decision:context-dogfooding` and `decision:shared-context-governance`).

Acceptance:

- Each product's canonical context lives either in a dedicated `{name}-context` repository or under `./context` relative
  to the application repository root.
- That context follows the applicable shared engineering conventions and `winter-canon` governance rather than an
  independent project-specific governance system, with adoption following `decision:shared-context-governance`.
- The application repository's agent entry point routes contributors and reviewers to that canonical context, including
  its development conventions, architectural boundaries, and verification guidance.
- The context is maintained with the product and used for implementation, code review, and architectural review; it does
  not exist only in planning files or private agent prompts.

## Install through Homebrew (product:homebrew-installation)

People can install the suite's released, public-facing CLI tools through the C5X Labs Homebrew tap without installing a
compiler or building from source. This applies across the platform, Workspace, and Fleet projects under
`decision:homebrew-channel`.

Acceptance:

- Each released product CLI has a documented `brew install c5xlabs/tap/<name>` installation on its supported targets;
  the platform executable is installed with `brew install c5xlabs/tap/winter`.
- Installing a product CLI also obtains its compatible shared `winter` launcher when needed, without a separate manual
  installation step or installing unrelated products.
- Installation uses prebuilt artifacts and requires neither a Rust compiler nor the existing Python Workspace CLI.
- The installed CLI's help and version work without workspace setup or a running product service.
- Homebrew can upgrade a released CLI, and uninstall removes only the Homebrew-managed installation without deleting
  workspace data or the user's shared version cache.
