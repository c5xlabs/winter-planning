---
name: document-dispatch-boundaries
depends-on:
  - prove-dispatch-boundaries
---

# Update contributor guidance for the implemented dispatch boundaries

Maintain `product:governed-context` for the implemented [System](../spec.md#system) and
[Interfaces](../spec.md#interfaces), including `engineering:fixture-only-dispatch` and
`engineering:legacy-adapter-boundary`. Follow `decision:context-dogfooding`,
`decision:launcher-shim/component-boundary`, and `decision:launcher-shim/manual-cutover`; this is contributor guidance,
not singleton cutover documentation.

## Change

- Update the application's canonical architecture/command-extension guidance for actual classifier, selection, cache,
  legacy setup, composition-root, and process-descriptor ownership.
- Describe injected environment/filesystem/synchronizer seams, route-specific write boundaries, and where modern and
  legacy errors are rendered.
- Update verification guidance with the implemented fixture cases and focused foundation-driver commands, including
  process/concurrency/cancellation coverage and production/sample isolation.
- Keep release installation, manual replacement/rollback procedures, Rust Workspace adoption, and future legacy
  retirement out of these contributor instructions.

## Acceptance

- Following the application's existing agent entry point reaches current dispatch architecture, extension touchpoints,
  and verification guidance under the existing canonical-context layout.
- Every referenced symbol, fixture, and command exists; focused test examples exercise their stated coverage without
  invoking installed winter, current Python Workspace, live components, or real selection/cache files.
- Guidance distinguishes read-only modern resolution from bounded legacy/uv writes, target from selected source, and
  shared execution from exclusive setup ownership without inventing new policy.
- Repository Markdown checks pass, and no duplicate planning contracts, cutover runbook, installer, or migration tooling
  is introduced.
