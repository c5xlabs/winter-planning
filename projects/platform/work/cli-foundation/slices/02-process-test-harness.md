---
name: process-test-harness
external: github:c5xlabs/winter#3
depends-on:
  - workspace-cli
---

# Add the bounded process harness and production regression suites

Implement `engineering:test-project`, `engineering:cli-contract-tests`, and the focused-test portion of
`engineering:local-check-loop`, proving `product:discover-help`, `product:identify-version`,
`product:inspect-without-setup`, `product:unsupported-invocation`, and `product:development-entrypoint` through the
built executable and checkout-specific development entry point. The contracts are the
[development driver and harness](../spec/development.md#domain), its
[command interfaces](../spec/development.md#interfaces), and [CLI replies](../spec/cli.md#interfaces).

## Change

- Implement shared invocation/observation fixtures in the harness crate, isolated runtime environments and directories,
  temporary-file stream capture, finite deadlines, readiness-triggered signaling, and bounded failure diagnostics.
- Give each driver-owned Cargo/tool subprocess its own process group; runtime fixtures inherit that group without
  detaching. Per-case guards own direct children, while driver cancellation and completion clean up the whole group.
- Implement configuration-specific builds, Cargo JSON executable discovery, expected-version metadata, and explicit
  service-test artifact/scope inputs. Preserve the build-toolchain environment separately from runtime isolation.
- Expose independently runnable unit, component, and service test commands with focused filters and incremental target
  reuse. Component tests call the platform facade; service tests launch the built CLI without PATH lookup.
- Add the driver's development `run` command and thin `scripts/winter-cli` entry point, selecting the checkout and
  pinned toolchain explicitly while preserving caller cwd/arguments and leaving the installed workspace `winter`
  untouched.
- Add production regression cases and marker-writing PATH decoys that prove unsupported product/sample invocations do
  not launch another executable, even when development variables are set.
- Add reply-write failure fixtures using the selected read-only output descriptor, exercising the real CLI's output and
  exit handling rather than treating application write errors as harness spawn failures.

## Acceptance

- Each test tier runs independently, and selecting a test-name filter does not require formatting/lint checks, deletion
  of artifacts, or a clean release build.
- Production executable tests assert stdout, stderr, status, help equivalence, package-derived version, unknown inputs,
  and unsupported groups outside a workspace using isolated HOME/XDG/temp locations.
- A help/version reply sent to a write-rejecting stdout exits `1` and produces an I/O diagnostic on writable stderr. An
  unsupported-invocation reply sent to a write-rejecting stderr also exits `1` even though its best-effort diagnostic
  cannot be delivered. Both cases use the normal finite deadline and process cleanup.
- Setting the sample-path variable or supplying matching product/sample PATH decoys cannot enable production dispatch;
  decoy markers remain absent and real user configuration/installations remain untouched.
- Cargo artifact selection matches package ID and binary target name, rejects missing/ambiguous executables, and passes
  absolute paths to tests; direct service-test execution without required driver inputs fails diagnostically.
- Harness tests cover stdin/capture, timeout, output-limit diagnostics, exit-before-readiness, readiness-triggered
  signals, and cleanup on failure. Driver cleanup reaches non-detaching fixtures after their test process terminates.
- Driver Cargo/tool children and runtime children have their specified finite deadlines; interrupt/termination paths
  perform owned-process cleanup without signaling the driver's own process group.
- Unit/component tests share the executable's production implementation and fixture builders rather than copying CLI
  behavior or requiring mock Workspace/Fleet implementations.
- Invoking the development wrapper from an unrelated cwd selects this checkout's version, preserves raw arguments, cwd,
  and streams, and reports CLI failure; a marker-writing `winter` decoy on PATH is not invoked. Setup requires no global
  alias, profile edit, or installation that would shadow the existing workspace tool.

## Out of scope

- Real sample modes and handoff assertions belong to `sample-executable` and `sample-handoff`; this slice provides their
  existing process/test seams without adding general product dispatch.
