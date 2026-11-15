---
name: production-process-contract
depends-on:
  - epic:cli-foundation
code:
  - winter/crates/cli/src/main.rs::main
---

# Extend the process boundary for production dispatch

Establish the shared request, environment, error, and process contracts from [Domain](../spec.md#domain) and the
[lease-descriptor flow](../spec.md#flows), serving `product:preserve-invocation`, `engineering:legacy-adapter-boundary`,
and `engineering:fixture-only-dispatch`. Keep the existing sample behavior under
`decision:launcher-shim/component-boundary`; no production command is registered yet.

## Change

- Promote `Process` and `UnixProcess` out of the development-only gate and extend `ExecRequest` with explicit
  environment, optional cwd, and owned inherited descriptors.
- Add the public modern and legacy request/environment/error contracts and injected `LegacySetup` boundary without
  implementing resolution or setup.
- Make `UnixProcess` use the request's native environment directly, never global environment reads, PATH discovery, or
  shell fallback.
- Preserve owned descriptors through replacement, clear their close-on-exec flags only immediately before final exec,
  and close them on replacement failure. Leave standard-stream descriptors unchanged.
- Inject the inherited environment from main for sample dispatch; supply its existing environment policy, inherited cwd,
  and no inherited descriptors while retaining its development-only registration.

## Acceptance

- Production and development configurations compile the process facade; production still rejects the sample command.
- Process-double tests inspect exact executable, native argv/environment, cwd, and descriptor ownership without faking a
  successful exec as a normal return.
- Direct adapter fixtures prove inherited cwd and explicit legacy cwd, unchanged streams, and no global-environment
  dependency. Embedded NUL and invalid executable format fail without a shell fallback.
- Descriptor fixtures preserve both open and closed fds 0/1/2, retain a supplied fd across successful exec, and observe
  its closure after failed replacement.
- Existing sample service tests retain their PID, raw-argument, environment-removal, stream, exit, and signal behavior.

## Out of scope

- Version resolution, legacy setup, and public command wiring belong to `resolve-role-versions`,
  `prepare-legacy-launch`, and `wire-launcher-dispatch`.
