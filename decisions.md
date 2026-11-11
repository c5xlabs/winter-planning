# Suite decisions

## Shared release policy baseline (decision:release-policy-baseline)

**Decider:** product owner and engineering

**Open.** Choose the remaining shared rules for compatibility promises, change notes, candidate promotion, and release
approval from Blizzard's process under `decision:release-versioning`; this blocks finalizing
`engineering:platform/release-automation/release-artifacts`. Define which rules are mandatory for initial releases and
which remain product-specific or are adopted later, without assuming Blizzard's cadence and support windows apply
everywhere.

## Shared CLI runtime baseline (decision:shared-cli-runtime-baseline)

Platform, Workspace, and Rust Fleet CLIs share the release target and minimum OS/libc policy under
`engineering:cli-runtime-baseline`, rather than choosing separate product baselines. Shared release enablement owns
settling the still-open minimum versions; product-specific provider prerequisites remain separate, and Foundation's CI
images do not become minimum supported release systems.

## Shared release capability (decision:shared-release-capability)

**Decider:** product owner and engineering

Platform owns `epic:platform/release-automation` as shared release enablement for Platform, Workspace, and Fleet, while
each product owns its artifact-specific integration and releases independently. Consumer release epics and milestones
declare their dependency using the existing planning conventions, not project-wide development blockers; existing
Blizzard releases continue until deliberately migrated.

## Release versioning (decision:release-versioning)

**Decider:** product owner and engineering

Use Semantic Versioning per independently shipped product/release unit, rather than a suite-wide number or a mandatory
version per executable, so release coordination follows actual delivery boundaries. CLI tools, services, daemons,
backends, and bundled web apps released together share the unit's version; split versions only for genuinely independent
releases, with compatibility specified separately and version rules defined in `engineering:release-versions`.

## Shared release governance (decision:shared-release-governance)

**Decider:** product owner and engineering

Establish shared package, versioning, and release governance in the new `c5xlabs/winter-context`, drawing selectively
from Blizzard's `initiative:fleet/release-engineering` rather than requiring each product to invent its own process.
Separate shared policy from product-specific tooling, cadence, and compatibility surfaces, so
`epic:platform/release-automation` can implement the applicable baseline with `cargo-dist` without importing Blizzard's
entire release system.

## Shared context governance (decision:shared-context-governance)

**Decider:** product owner and engineering

Establish a fresh `c5xlabs/winter-context` as shared engineering governance for Winter and Blizzard applications under
`winter-canon`, separating common conventions from language- and application-type-specific rules and selectively
curating content rather than transferring or wholesale-merging the existing context repositories. Adoption is
incremental: establish the minimum shared baseline needed by `epic:platform/cli-foundation` first, keep existing
consumers on their current governance until deliberately migrated, and sunset `paul-gross/winter-context` and
`blizzard-context` only after their consumers have moved.

## Context dogfooding (decision:context-dogfooding)

Every project uses the suite's own `winter-canon`-governed context structure for development and review rather than
separate, ad hoc setup and extension documents, so the suite exercises the conventions it supports. Each application's
own canonical context may live in a dedicated `{name}-context` repository or its root-relative `./context`.

## Homebrew channel (decision:homebrew-channel)

The suite's initial Homebrew channel is `c5xlabs/tap`, backed by `c5xlabs/homebrew-tap`, with each CLI installed through
`brew install c5xlabs/tap/<name>` using prebuilt release artifacts rather than compiling on the user's machine. After
verified assets are published, automation with narrowly scoped credentials submits formula-update PRs for maintainer
approval, keeping installation updates explicitly reviewed while the release process is established.
