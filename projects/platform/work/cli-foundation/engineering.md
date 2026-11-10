# Winter CLI foundation engineering requirements

For `epic:cli-foundation`.

## What already holds

The platform's implementation language is constrained by `engineering:rust-product-clis`; this epic establishes
the Rust project, test facilities, and delivery checks rather than choosing another implementation language.

## Reproducible development setup (engineering:reproducible-setup)

The repository pins its Rust toolchain and dependency resolution and documents the operating-system build prerequisites.
Setup must not depend on the current Python CLI, an installed Winter product, or machine-local configuration. Serves
`epic:cli-foundation`.

Acceptance:

- A fresh checkout builds with the documented prerequisites and checked-in toolchain and dependency locks.
- Building and testing do not change the dependency lockfile or require credentials for Winter services.
- Build output remains untracked and local development does not replace a command on the user's PATH.
- The product's governed context documents the prerequisites and exact build, run, and verification commands; setup does
  not depend on private contributor notes.

## Focused local feedback (engineering:local-check-loop)

The repository exposes documented commands for formatting, linting, compilation checks, focused tests, and the routine
verification suite, with publishing and package checks available separately. Routine development reuses incremental
build artifacts rather than requiring clean release builds. Serves `epic:cli-foundation`.

Acceptance:

- One documented aggregate command runs the required local checks and exits unsuccessfully when any check fails.
- A contributor can run a selected test without running every check or deleting build artifacts first.
- After toolchain and dependency bootstrap, the local verification loop runs without network access or live services.
- Check mode reports formatting and lint failures without silently rewriting source files.

## Automated quality gates (engineering:automated-checks)

Continuous integration runs the same formatting, linting, compilation, and test checks required locally on code changes.
The baseline includes rustfmt and Clippy, with compiler and Clippy warnings treated as failures. Serves
`epic:cli-foundation`.

Acceptance:

- A formatting violation, a compiler or Clippy warning, and a failing test each cause an unsuccessful check result.
- Local and CI entry points use the pinned toolchain and locked dependencies and do not maintain divergent check lists.
- A CI failure identifies the failed check and retains its diagnostic output.
- The lint configuration is checked in and covers production code, test code, and enabled features/targets in the
  supported build matrix; exclusions require a narrow scope and an explanation.
- Repository-owned Markdown and release/automation files have documented, automated format or lint checks appropriate
  to their file types, not only checks for Rust source.

## Dedicated test project (engineering:test-project)

The repository includes a dedicated Rust test project/harness with shared fixtures and separately runnable unit,
component, and service-test suites. Unit tests may remain beside the Rust code they exercise, but the test project
provides the common tier definitions and entry points. Serves `epic:cli-foundation`.

Acceptance:

- Unit tests exercise individual functions or types; component tests exercise cooperating platform modules through
  their declared interfaces without launching the CLI process.
- Service tests launch the built CLI executable as a subprocess; the term does not imply a daemon or Fleet service.
- Each tier has an independently runnable command, and the aggregate local and CI checks include all three tiers.
- Shared fixture/configuration builders live in test support rather than being copied into each suite; production code
  does not depend on the test project.
- Test targets consume the same production implementation used by the executable, not a parallel implementation of its
  behavior.

## Executable-level regression coverage (engineering:cli-contract-tests)

Automated tests invoke the newly built platform executable and assert stdout, stderr, and exit status independently.
They run in isolated directories and environments, without invoking the installed Winter command. Serves
`product:discover-help`, `product:identify-version`, `product:inspect-without-setup`, and
`product:unsupported-invocation`.

Acceptance:

- Tests cover bare invocation, help, version, unknown options, and unsupported product groups, including execution
  outside a workspace.
- Test fixtures can contain stand-in product executables and prove the foundation does not invoke them.
- Tests verify the executable's version against package metadata rather than maintaining an independently hardcoded
  version expectation.
- Tests do not read or mutate the user's real home configuration, workspaces, or installed executables.
- The service-test harness accepts an explicit built-executable path and supplies isolated working directories,
  environment variables, input, and configuration for each invocation; it does not locate the CLI through PATH.
- Every spawned process has a finite test timeout; on failure or timeout the harness captures diagnostics and terminates
  and reaps the processes it owns.
- The harness allows controlled external collaborators to be supplied by later tests. This epic tests help, version,
  errors, and the development sample without requiring a mock Workspace/Fleet implementation or installed-product
  discovery.

## Development-only process boundary (engineering:sample-process-boundary)

The sample is a separate Rust test/development project, not behavior linked into the platform executable. Its explicit
development registration exercises the process-launch boundary that the later launcher can extend. Serves
`product:development-sample` under `decision:cli-foundation/development-sample`.

Acceptance:

- The dev harness supplies the sample executable explicitly; the demonstration does not scan PATH, resolve workspace
  version pins, download components, or invoke the installed `winter` command.
- The platform's production modules do not depend on the sample implementation; sample outputs come from the launched
  process, not a reimplementation inside the CLI.
- Component tests exercise the process-launch interface with controlled doubles, and service tests launch the built
  development CLI and actual Rust sample to verify arguments, working directory, streams, and exit status.
- A release-mode packaging check proves that the development registration is disabled and the sample binary is absent,
  including when test environment variables or matching PATH entries are present.
- The sample project is excluded from package publishing and Homebrew formula contents; local demonstration commands
  do not install or replace either executable on the user's PATH.

## Command and dependency boundaries (engineering:command-boundaries)

Argument parsing and process-level output/exit handling are separated from command behavior, with application wiring at
an explicit entry point. A command does not construct hidden cross-command dependencies or access a global service
locator. Serves `epic:cli-foundation`.

Acceptance:

- The architecture names the responsibilities and dependency direction between CLI entry, command behavior, and shared
  facilities, with one established pattern for registering a command.
- Domain behavior can be tested without invoking the process-level entry point when such behavior exists.
- The foundation does not add empty product modules, placeholder services, or an extensibility framework for commands
  outside this epic; its structure supports growth without implementing those capabilities.
- Allowed dependencies and forbidden edges are stated at the module/package level, including the production/test
  boundary; dependency cycles and production imports of test support are rejected by automated checks.
- A prohibited cross-boundary dependency causes the architectural check to fail, with a diagnostic naming the offending
  modules or packages; documentation alone is not the enforcement mechanism.

## Agent-facing architecture guidance (engineering:extension-guidance)

The code repository's agent entry point routes contributors to its architecture, command-extension procedure, and
verification commands. The guidance records project-level defaults rather than asking each feature author to choose
them again. Serves `product:governed-context`.

Acceptance:

- The canonical context uses one of the layouts in `product:governed-context` and declares its `winter-canon` governance
  through the repository's normal context discovery chain.
- Guidance establishes package/module ownership, dependency wiring, argument-parsing conventions, error propagation,
  stdout/stderr ownership, version metadata, and test placement.
- The extension procedure identifies the files and checks involved in adding a command, using the foundation's actual
  structure rather than an unrelated example or a speculative product implementation.
- Shared-contract and dependency-boundary changes are identified as architectural decisions; command-local behavior
  within the established structure does not require an additional project-level decision.
- A verification matrix maps the promised CLI behavior and engineering checks to concrete commands a contributor or
  agent can run.
- Repository-owned Rust coding standards and architectural invariants are discoverable by code and architecture reviews
  through the same routing chain used during implementation; they do not live only in a reviewer prompt or this planning
  repository.
- The verification matrix distinguishes unit, component, service, lint, architecture, build, and release/package checks
  with concrete commands and stable method identifiers; it lists only methods available in the code repository.
- The guidance documents how to check a proposed boundary change and how to verify that ordinary changes stay within
  the existing boundaries; reviewers need not invent project policy for each change.

## Verified release artifacts (engineering:release-artifacts)

The release workflow builds versioned prebuilt archives from a recorded source revision and publishes them with
checksums after the required checks pass. Artifact targets and naming follow
`decision:cli-foundation/release-targets`. Serves `product:publish-platform-release`.

Acceptance:

- A documented version/tag operation triggers or prepares the release; the tag, package metadata, executable version,
  and artifact identifiers agree, and an inconsistent version fails the release checks.
- Published artifacts have checksums, contain the platform executable and required distribution material, and exclude
  development caches, credentials, and test-only executables.
- A release smoke check extracts the archive and runs help and version on each supported execution target; compilation
  alone is not treated as proof that a packaged binary runs.
- The release workflow records the source revision and check results and does not silently replace an already published
  version with different artifacts.
- Ordinary build/test checks do not need publishing credentials; publishing permissions are scoped to the release path.

## Homebrew distribution (engineering:homebrew-distribution)

The Homebrew formula installs the matching verified prebuilt artifact for each supported Homebrew target, with its
checksum and version recorded in the formula. The tap and formula are owned through
`decision:homebrew-channel`. Serves `product:homebrew-installation`.

Acceptance:

- A documented release step updates the tap formula to the published version and checksums; a formula never references
  artifacts that have not been published.
- Formula checks and installation smoke tests verify help, version, upgrade, and uninstall using a controlled Homebrew
  environment, not the developer's live installation.
- Formula installation downloads prebuilt artifacts and requires no Rust compiler, Python runtime, or Winter product
  dependency.
- The code repository documents release prerequisites, the Homebrew update procedure, and the supported target matrix
  so publishing is repeatable rather than a collection of machine-local steps.
