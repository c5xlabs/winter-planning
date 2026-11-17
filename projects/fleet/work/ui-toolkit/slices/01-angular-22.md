---
name: angular-22
external: github:paul-gross/blizzard#2255
depends-on:
  - epic:storybook
code:
  - blizzard/web/projects/fleet/.storybook/main.ts::config
---

# Upgrade the Angular toolchain

Moves the web workspace to [platform.md](../spec/platform.md#system)'s compatible version set before either adoption,
under `decision:ui-toolkit/angular-22-first`. This prepares `product:full-adoption` without changing form behavior.

## Change

- Upgrade Angular, CDK, packaging, TypeScript, lint, test, and workshop integrations together.
- Preserve the documented Vite split, strict-install policy, and generated-client workflow.

## Acceptance

- A strict `npm ci` succeeds; dependency resolution does not require a committed relaxed-peer setting.
- `web:lint`, `web:typecheck`, `web:unit-test`, and `web:story-render` pass on the upgraded workspace.
- `web:bundle-composition` passes and the existing initial-bundle budgets remain unchanged.
