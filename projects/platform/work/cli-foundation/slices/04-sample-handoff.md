---
name: sample-handoff
depends-on:
  - process-test-harness
  - sample-executable
---

# Wire the feature-gated sample handoff and prove replacement

Deliver `product:development-sample` and `engineering:sample-process-boundary` through the
[sample process interface and flow](../spec/sample.md), [raw-tail registration](../spec/cli.md#flows), and
[development demonstration command](../spec/development.md#interfaces). The permanent default-off `dev-sample` feature
isolates development behavior; it is not a temporary rollout flag to remove.

## Change

- Add the single feature-gated sample registration/help annotation and public process seam. Preserve every raw argument
  after the initial `sample` token, including option-like and non-UTF-8 values and literal delimiters.
- Read the sample path only after selecting a development sample request; validate the injected native path and map
  missing/relative paths or replacement errors to the specified launcher reply.
- Implement direct Unix `execve` with owned native argv/environment storage, removing only the sample-path variable and
  preserving PID, cwd, streams, and remaining environment without shell fallback or PATH discovery.
- Add controlled process-double component tests and real development service tests using the independently built sample.
- Implement `cargo dev sample -- …`: build artifacts, inject the absolute sample path, then replace the driver with the
  CLI instead of adding a waiting parent or signal-forwarding layer.

## Acceptance

- Separate Cargo invocations and target directories build/test production and development configurations; all sample
  registration and platform replacement wiring are compiled only with the default-off feature enabled.
- Production help and parsing continue rejecting sample dispatch despite environment variables and PATH decoys;
  development help advertises it and forwards its raw tail without clap interception or delimiter removal.
- Component doubles observe the exact launch request and controlled failure; service tests prove successful replacement
  by matching the originally spawned CLI PID with the sample's reported PID.
- Real sample tests verify option-like/non-UTF-8 arguments and executable paths, cwd/environment preservation, removal
  of the sample-path variable, stdin/stdout/stderr bytes, exit statuses, and signal termination after readiness.
- Missing, empty, relative, nonexistent, non-executable, and executable-invalid-format targets fail diagnostically;
  executable text without a shebang does not execute its marker-writing body through a shell.
- Tests distinguish failed replacement from a sample that starts and exits unsuccessfully; timeout/failure paths leave
  no owned fixture running.
- The demonstration requires no network after bootstrap, installed Winter component, Python CLI, or product service;
  exec failure is reported, and successful execution leaves no supervising driver or launcher parent.

## Out of scope

- General product dispatch, invocation-aware product help, discovery, and version-selection policy remain follow-on
  work; this slice introduces no compatibility bridge or product installation.
