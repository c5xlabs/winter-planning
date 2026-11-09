# Mission

The platform is what every C5X Winter product stands on: the `winter` command a person installs once, the way each
product arrives and upgrades, and the contract an extension written by anyone plugs in through. It serves everyone who
uses any product of the suite, and the people who extend it.

## Scope filter

A proposed feature belongs to the platform when more than one product needs it, or when a person meets it before they
reach any one product: installing, upgrading, choosing a version, or extending the suite.

## Success criteria

1. **Installed in one step.** `persona:application-architect` installs the `winter` command once and adds Workspace,
   Fleet, or both from it, with no toolchain to set up first.
2. **Each workspace keeps its own version.** Two workspaces on one machine pin different versions of the suite, and
   upgrading one never changes the other.
3. **Extensions in any language.** `persona:harness-engineer` writes an extension in the language they prefer, and it
   keeps working across releases because the contract it speaks is versioned.
4. **One command shape.** Whichever product a person is using, commands are found, named, and answered the same way, so
   learning one product teaches the shape of the others.

## Non-goals

- **Product features.** What a workspace or a fleet does belongs to its product; the platform only carries a person to
  it.
- **In-process plugins.** Extensions run as separate programs behind the contract, never loaded into a product's own
  process.
- **A build-from-source requirement.** Building the suite from source is for people changing its core, never a step for
  using it.
