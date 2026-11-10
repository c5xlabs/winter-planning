# Winter CLI foundation requirements

For `epic:cli-foundation`.

## Discover the supported interface (product:discover-help)

A person can ask the executable what it does and how to use the interface it actually supports. Invoking it without
arguments also shows help (`decision:cli-foundation/bare-invocation`).

Acceptance:

- `winter --help` identifies the Winter platform command, shows usage, and describes its help and version options.
- `winter` without arguments displays the same help as `winter --help` and exits successfully.
- Help does not present Workspace, hub, runner, or installation commands as available functionality.
- Released help does not advertise the development sample.
- Asking for help exits successfully.

## Identify this executable (product:identify-version)

A person can identify the platform executable they invoked, without confusing its version with a Workspace or Fleet
component's version.

Acceptance:

- `winter --version` reports the command name and the version of that executable, then exits successfully.
- The reported version does not depend on the current directory or another installed Winter component.

## Inspect without setup (product:inspect-without-setup)

A person can use help and version before adopting either Winter product, and can try the platform executable alongside
an existing Winter installation without changing that installation.

Acceptance:

- Help and version work outside a Winter workspace, without workspace configuration or installed Winter components.
- Running the executable does not create a workspace, install components, or alter the existing `winter` command.

## Explain unsupported invocations (product:unsupported-invocation)

A person or agent attempting an unsupported invocation receives a failure that points them toward the supported
interface instead of silently running another tool.

Acceptance:

- An unknown option or command produces an explanatory error, points to help, and exits unsuccessfully.
- Invocations such as `winter workspace`, `winter hub`, and `winter runner` fail as unsupported commands, even when
  another Winter CLI or a matching product executable is installed.
- An unsupported invocation does not start another command, prompt to install a component, or change workspace files.
- A released executable rejects `winter sample`, even when a sample executable exists on PATH.

## Demonstrate a separate component (product:development-sample)

A contributor can run a dev-only Rust sample through the platform CLI and observe a real handoff to a separate
executable (`decision:cli-foundation/development-sample`). The demonstration validates the module boundary without
installing a product or using Homebrew.

Acceptance:

- A documented development setup builds the platform CLI and Rust sample, then runs it through `winter sample …`.
- The sample runs in a separate process and exposes the received arguments and working directory, along with controlled
  stdout, stderr, and exit status, so the contributor can see that the handoff preserved the invocation.
- A missing sample executable or a sample command failure produces an unsuccessful invocation with useful diagnostics.
- The demonstration requires no existing Python CLI, Workspace/Fleet implementation, product installation, or network
  access after development dependencies have been fetched.
- The sample command and executable are absent from release archives, the Homebrew installation, and normal installed
  help; the sample is not a separately distributed product.

## Publish an identifiable release (product:publish-platform-release)

A maintainer can publish a versioned platform executable that users can identify and obtain independently of the source
checkout. The initial supported targets are settled by `decision:cli-foundation/release-targets`.

Acceptance:

- A documented release procedure publishes prebuilt executables for the supported operating-system and CPU targets.
- A downloaded release reports the version under which it was published.
- A maintainer can identify the source revision and verification results associated with the release.
- Publishing the platform does not require releasing or installing Workspace or Fleet.
