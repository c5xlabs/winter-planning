# Mission

The Winter platform houses the shared capabilities that let separate tools such as Winter Workspace and Winter Fleet
exist and work together. It serves the people building and extending those tools, the individuals bringing them to their
projects, and the teams or organizations governing their use.

Its contribution to [Winter's mission](../../../charter/mission.md) is to give those tools a common foundation for
development, distribution, operation, integration, and governance while keeping each independently useful and
developer-owned. The `winter` launcher, installation and version selection, and extension contracts are parts of that
foundation, not the limit of the platform's purpose.

## Scope filter

A proposed feature belongs to the platform when it supplies a reusable foundation that enables Winter tools to be built,
distributed, operated, extended, integrated, or governed. It must serve an identified product or user need, preserve
developer ownership, and avoid tying the tools to one project, provider, or hosted service. Behavior specific to a
tool's own job belongs in that tool; sharing its foundation does not move its workflows into the platform.

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
5. **Tools they can carry.** `persona:harness-engineer` can adapt the open-source platform and reuse their extensions
   across projects without rebuilding them around each project's choice of Winter products.
6. **Shared tools, governed use.** `persona:application-architect` can make selected versions and extensions part of a
   team's development setup, while `persona:harness-engineer` can use those same tools independently on another project.

## Non-goals

- **Owning every automation capability.** Workspace and Fleet own their development workflows; the platform supplies
  shared capabilities rather than duplicating their behavior.
- **In-process plugins.** Extensions run as separate programs behind the contract, never loaded into a product's own
  process.
- **A build-from-source requirement.** Building the suite from source is for people changing its core, never a step for
  using it.
