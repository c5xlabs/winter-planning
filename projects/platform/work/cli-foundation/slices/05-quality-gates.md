---
name: quality-gates
depends-on:
  - sample-handoff
---

# Enforce architecture and run the shared local/CI gates

Complete `engineering:command-boundaries`, `engineering:automated-checks`, and `engineering:local-check-loop` through
the [dependency checker](../spec/system.md#interfaces) and
[development command/check matrix](../spec/development.md#interfaces). Preserve `decision:release-delivery-separation`:
these gates verify source development, not release readiness.

## Change

- Implement the Cargo-metadata workspace-member/dependency allowlist over all declared normal, dev, build, optional,
  renamed, and target-specific workspace edges; retain Rust visibility and Cargo's cycle checks.
- Add architecture checker self-tests and actionable diagnostics identifying forbidden edges; expose
  `cargo dev
  architecture` without source-scanning machinery.
- Complete format/check-only and explicit formatting-write commands, Clippy and compiler warning enforcement, and
  Markdown, configuration, workflow, and bootstrap automation checks using repository-local pinned tools.
- Include both development scripts in formatter/shell checks, including the extensionless `scripts/winter-cli` wrapper.
- Run actionlint with ShellCheck/pyflakes integrations disabled explicitly so installed optional tools cannot change the
  check set. Missing tool/cache prerequisites fail instead of triggering routine installation/downloads.
- Implement the single `cargo dev check` operation list over both configurations, all Rust targets, all three test
  tiers, architecture, formatting, and linting; reuse it unchanged in the specified CI host matrix.
- Preserve toolchain/build environment handling, separate incremental targets, bounded Cargo/tool execution, and
  retained diagnostics in the automated path.

## Acceptance

- Injected forbidden dependencies fail architecture checks, including disabled optional, renamed, dev, build, and
  target-specific edges; diagnostics name the offending packages and dependency kind/condition.
- Default and sample-enabled source/test targets pass the same warning-enforcing checks; a compiler warning, Clippy
  warning, format violation, lint failure, or failing test makes the aggregate command fail.
- Ordinary format/check commands do not rewrite files; only the explicit `--write` mode does so. Repository Markdown,
  formatter/linter configuration, workflow YAML, and bootstrap shell automation are included in their declared checks.
- After explicit bootstrap, local checks run offline without live services; focused test commands still reuse artifacts
  rather than invoking the full aggregate sweep.
- Installing unrelated ShellCheck or pyflakes binaries does not change actionlint's behavior; mismatched/missing pinned
  tool versions and absent cached prerequisites fail with setup guidance.
- CI invokes the same aggregate operation list on all specified hosts, identifies failed checks and retains their
  diagnostics, with no release credentials, artifact publication, Homebrew installation, or installed Winter dependency.

## Out of scope

- Publication, signing, release-target baselines, and first-release readiness belong to `epic:release-automation`;
  development CI host images do not establish supported release OS/libc minima.
