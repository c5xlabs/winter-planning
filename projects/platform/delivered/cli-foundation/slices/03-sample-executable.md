---
name: sample-executable
external: github:c5xlabs/winter#4
depends-on:
  - workspace-cli
---

# Implement the independent development sample

Build the observable Rust executable for `product:development-sample` and `engineering:sample-process-boundary`, using
the [sample modes and byte-level records](../spec/sample.md#interfaces) within the
[workspace dependency boundary](../spec/system.md#system). The sample is development-only and non-publishable; it does
not implement launcher behavior or introduce a distributed product.

## Change

- Implement standalone inspection, help, stdin-copy, stdout/stderr emission, requested-exit, self-termination, and wait
  modes with the specified argument/error behavior.
- Report PID, native argument/cwd bytes, and only the named environment probes; use hex rather than lossy UTF-8.
- Flush readiness before waiting or self-termination, resetting and unblocking the required signals as specified.
- Add inline sample unit tests without linking the sample implementation into the platform or harness libraries.

## Acceptance

- The sample builds independently, has no workspace package dependency, and remains marked `publish = false`.
- Inspection returns the documented JSON fields, includes the actual mode/argument vector, preserves non-UTF-8 bytes,
  and represents absent probe variables as `null` without dumping the full environment.
- Help and malformed mode invocations follow the specified output/status behavior; inspection operands remain literal,
  including `--`, while an initial bare delimiter is invalid usage.
- Stdin-copy and emission modes produce the specified stream bytes; requested-exit mode flushes its diagnostic and
  returns the requested status, distinct from a launch failure.
- Self-termination and wait modes flush their readiness marker and respond to the specified default, unblocked signals.
- Sample unit tests run independently of the production CLI, and the production CLI still neither advertises nor invokes
  the sample.

## Out of scope

- Launcher registration, replacement, and demonstration tooling belong to `sample-handoff`; no arbitrary-command mode or
  future product-discovery protocol is added.
