# Mission

The platform is what every Winter product stands on: the `winter` command a person installs once, the way each product
arrives and upgrades, and the contract an extension written by anyone plugs in through. It serves everyone who uses any
product of the suite, and the people who extend it.

## Scope filter

A proposed feature belongs to the platform when more than one product needs it, or when a person meets it before they
reach any one product: installing, upgrading, choosing a version, or extending the suite.

## Success criteria

These outcomes serve `persona:senior-se`, `persona:harness-engineer`, and `persona:application-architect`.

1. **Installed in one step.** They install Workspace, Fleet, or both with the shared `winter` entry point, following
   `product:homebrew-installation`, with no toolchain or manual assembly to set up first.
2. **Each workspace keeps its own version.** They pin two workspaces on one machine to different versions of the suite,
   and upgrading one never changes the other.
3. **Extensions in any language.** They write an extension in the language they prefer, and it keeps working across
   releases because the contract it speaks is versioned.
4. **One command shape.** Whichever product they use, commands are found, named, and answered the same way, so learning
   one product teaches the shape of the others.

## Non-goals

- **Product features.** What a workspace or a fleet does belongs to its product; the platform only carries a person to
  it.
- **In-process plugins.** Extensions run as separate programs behind the contract, never loaded into a product's own
  process.
- **A build-from-source requirement.** Building the suite from source is for people changing its core, never a step for
  using it.
