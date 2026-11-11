---
name: governed-contributor-context
external: github:c5xlabs/winter#7
depends-on:
  - quality-gates
---

# Publish the governed contributor and review guidance

Complete `engineering:extension-guidance` and the documentation portions of `engineering:reproducible-setup`,
`engineering:local-check-loop`, and `engineering:test-project`, serving `product:governed-context` and
`product:development-sample` and `product:development-entrypoint`. Use the
[agent-facing artifacts and method IDs](../spec/development.md#interfaces),
[actual architecture](../spec/system.md#domain), and `decision:shared-context-governance`; canonical context placement
remains operator-selected, not automated by the application.

## Change

- Route the application agent entry point through the existing discovery chain to its manually selected canonical
  context and applicable shared engineering/`winter-canon` governance.
- Document actual package/module ownership, dependency wiring, parser/entry-point conventions, error/output ownership,
  executable identity, Rust coding standards, and test placement for implementation and both review types.
- Describe command extension against the implemented registration site and facade; distinguish routine command-local
  changes from architectural/shared-boundary decisions and their checks.
- Publish the verifiability matrix with the specified stable method IDs and working build, lint, architecture, unit,
  component, service, aggregate, and sample commands; distinguish commands, manual methods, and tools.
- Document prerequisites, explicit bootstrap, offline incremental feedback, focused tests, tier definitions, and sample
  demonstration using the commands that now exist, not private notes or speculative release procedures.
- Use the distinct `winter-cli` development entry point for manual checkout runs and keep installed `winter` workspace
  commands separate; document that the helper does not rename the public executable or require global shell changes.

## Acceptance

- Following the repository agent entry point reaches the same canonical standards and architecture guidance for
  implementation, code review, and architectural review under the operator-selected allowed context layout.
- Shared conventions resolve through existing workspace-managed governance without an application-owned pin, context
  fetch/update mechanism, automatic placement, or independent governance system.
- Every matrix method resolves to a documented implemented command and its correct test tier/check/tool role; focused
  examples run without invoking unrelated gates, and no unavailable release method is listed.
- Following the documented fresh-checkout setup, production help/version run, and sample demonstration succeeds without
  replacing PATH executables or requiring Python, product installation, live services, or release credentials.
- The extension procedure identifies real code touchpoints and verification commands, while boundary-change guidance
  names the architecture check and shared rules reviewers must apply rather than inventing new policy per feature.
- The application-owned context describes the delivered foundation rather than copying planning documents or adding
  placeholder Workspace/Fleet modules; its Markdown passes the repository's checks.

## Out of scope

- No governance repository transfer/sunset, context-placement automation, product implementation, release workflow, or
  new decision record is introduced by this documentation slice.
