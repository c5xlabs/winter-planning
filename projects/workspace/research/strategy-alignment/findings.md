# Workspace strategy alignment findings

## Baseline and authority

The baseline is a Rust `winter-workspace` replacement, independently usable and reached through the platform's
`winter workspace` entry point, with no Python delegation or first-party Python runtime requirement. Current Workspace
planning and the explicit no-Python instruction supersede earlier migration suggestions; rough parity concerns
observable capabilities and safety, not Python internals, old command spellings, or the in-process plugin API.

Current authority comes from [Workspace decisions](../../decisions.md) and
[Workspace product requirements](../../requirements.md), especially `decision:workspace/rust-replacement`,
`decision:workspace/rough-parity`, and `decision:workspace/requirements-first`. The
[project engineering requirements](../../engineering.md) record cross-cutting invariants; capability requirements remain
with their epics. No Workspace specs or slices are established by these findings.

The earlier paper, *Winter platform — strategic direction*, was examined across its complete product and engineering
sections, §§3–10, including the whole port roadmap after Phase 0. This summary carries only technical and product
findings intended for public planning; it does not make that earlier direction an additional requirements authority.

## What survives the earlier direction

- §3, **Product structure**: independently usable Workspace and Fleet, separate product code, no Workspace service
  runtime, and cross-product communication through public contracts rather than shared implementation.
- §4, **Command tree**: one platform entry point and product-owned commands. Its example groups and flags are
  illustrative, not a frozen Workspace CLI. Platform-to-product argument preservation is distinct from forwarding
  arbitrary flags to a swappable service backend.
- §5, **Setup and bootstrapping**: a workspace owns configuration and project context instead of a fork of the tool;
  creation differs from convergence. Progressive context navigation must retain eager safety obligations for harnesses
  that cannot execute a session-start hook.
- §6, **Distribution, versioning, and side-by-side workspaces**: prebuilt normal installation, independent workspace
  versions, immutable released cache entries, and separation of released artifacts from mutable source builds. Current
  Platform decisions settle the XDG cache; current release governance settles independent release units.
- §7, **Extension model**: external executable contracts, version checks, structured results, and no in-process code
  loading. Built-in providers may compile with Workspace but must satisfy the same Winter-owned semantics as external
  adapters; adding an external provider must not require rebuilding Workspace.
- §8, **Implementation language**: Rust for public CLIs. For Workspace, the current replacement decision additionally
  excludes Python obligations for first-party behavior, including providers; this does not choose runtimes for
  user-owned external programs.
- §9, **What the code looks like**: typed domain/refusal/error models, constructor injection, an explicit composition
  root without a DI container, narrow collaborators, and replaceable I/O adapters. Example types, libraries, and wiring
  are not a Workspace spec.
- §10, **Port roadmap**: executable contract observations and safety scenarios remain useful, particularly for
  destructive operations. The historical requirement to first freeze behavior in Python, forward unfinished commands to
  Python, run mixed implementations, and retire Python only at the end is invalidated by the replacement baseline.

The roadmap's proposed crate split, Git library choices, async policy, dashboard library, and phase ordering are not
accepted implementation constraints. Legacy source and observations can inform fixtures without importing Python types
or requiring Python in the Rust verification loop.

## Existing engineering ownership

Suite [engineering requirements](../../../../engineering.md) already own `engineering:rust-product-clis` and
`engineering:release-versions`. Suite [decisions](../../../../decisions.md) own shared context, release governance, and
Homebrew policy. They are consumed, not redefined at Workspace project scope.

The [component-runtime engineering requirements](../../work/parallel-workspaces/component-runtime/engineering.md)
already own reproducible builds, local/CI checks, test tiers, isolated executable fixtures, invocation fidelity, output
discipline, and governed contributor guidance. Project invariants complement these records rather than duplicating their
checklists. Parity validation, migration, context navigation, and release acceptance retain their own bounds.

The existing engineering context contributes principles rather than Python tooling:

- `winter-context:/architecture/system-architecture.md`, **Core tenet: interchangeability**: Winter defines backend
  operations; arbitrary backend argument passthrough defeats substitutability.
- `winter-context:/architecture/dependency-injection.md`, **Rule** and **When app config IS the right injection**:
  explicit collaborators, module-shaped configuration, and bounded loader/lifecycle exceptions.
- `winter-context:/architecture/repository-pattern.md`, **Layout** and **What stays out**: isolate external I/O;
  distinguish reads from domain-significant mutations; keep cross-system orchestration outside individual adapters.
- `winter-context:/architecture/domain-modeling.md`, **Parameters outside the domain**, and
  `winter-context:/architecture/error-handling.md`, **Rule**: domain types where justified, explicit failures, and
  handling only where recovering, transforming, or reporting.
- `winter-context:/architecture/subprocess.md`, **Rule**: shell-free execution by default, with an explicit
  operator-owned exception whose environment-expansion exposure remains documented rather than claimed safe by
  substitution checks.
- `winter-context:/standards/testing.md`, **Assertion patterns by layer** and **Convention tests**: seam-based fakes,
  real-I/O evidence where external formats can drift, and self-tested architectural checks.
- `winter-context:/architecture/winter-cli.md`, **Tracing**, and `winter-context:/verification/winter.md`,
  **winter:manual-tracing**: preserve context across parallel and nested work, exclude sensitive span content, prevent
  invocation context entering managed service launches, and bound export shutdown without changing command outcomes.
- `winter-context:/verification/winter.md`, destructive-operation methods: verify actual state and filesystem effects in
  disposable scopes, not only process status. A source override is not a sandbox, and preflight refusal guarantees do
  not imply rollback after an external operation partially executes.

Python-specific pytest, Ruff, Pyright, container, sentinel, and import conventions do not become Rust requirements.
Shared Rust conventions and available verification methods belong in governed application context. Neither historical
Python startup measurements nor exporter timing thresholds establish a new Rust budget.

## Platform boundaries still requiring agreement

The [Platform foundation specs](../../../platform/work/cli-foundation/spec/system.md) define a planned Rust foundation
and a development-only sample process boundary, not a completed product plugin API. The
[launcher-shim decisions](../../../platform/work/launcher-shim/decisions.md) establish separate product executables but
leave `decision:platform/launcher-shim/component-discovery` and `decision:platform/launcher-shim/version-selection`
open. Discovery, role/protocol proof, precedence, workspace pin schema, root discovery, and unpinned selection must not
be independently invented by Workspace.

Invocation-aware help is promised by `product:platform/launcher-shim/invocation-aware-help`; its production metadata
transport is not the sample-path variable. Current shim requirements report missing components without downloading them
and exclude building source overrides. Historical generic PATH dispatch, automatic downloads, and source rebuilds are
therefore not current shim contracts.

Platform's mission assigns it the shared extension contract, but the inspected Platform work does not yet define that
contract. Platform and Workspace must establish ownership of common protocol/versioning rules versus Workspace provider
actions and capability binding before freezing either side; these findings do not select an ABI or wire schema.

Selecting a development checkout against an existing workspace remains a source-override need from the earlier
direction, not delivered shim functionality. Its owner, component selection/build behavior, and separation from released
caches need an explicit decision. Any such override must continue to make clear that changing the executable does not
isolate its target workspace.

Workspace's open `decision:workspace/command-vocabulary` and `decision:workspace/configuration-transition` retain their
stated ownership. `engineering:cli-runtime-baseline` makes Platform's release target and minimum OS/libc decisions
shared across Rust CLIs; Foundation's CI images are not minimum supported systems.
`decision:workspace/supported-systems` now owns only provider compatibility and external prerequisites.

`decision:workspace/observability-shutdown` owns the exporter shutdown bound, and
`decision:workspace/development-component-selection` owns the development-selection workflow. Command-latency targets,
benchmarks, and optimization are deferred rather than blocking parity. Shared release enablement remains independent of
ordinary development, with artifact integration owned by `epic:workspace/rust-adoption/workspace-release`.
