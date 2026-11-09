---
name: cutover
depends-on:
  - narrowing
---

# Cutover

Takes the old path apart once the owner declares cutover. Delivers `engineering:container-only` and the end-state form
of `engineering:rules-follow-wiring`; waits on `decision:adopt-di/declared-cutover`.

## Change

- The hand-written builders are removed first.
- Each bag, and the bridges that served it, is removed the day its last consumer narrows.
- The gates admit only the container: no bag type in a signature, no process collaborator constructed outside a
  provider.
- `bzh:dependency-injection` and `bzh:narrow-seams` take their end-state form, with the reasoning of
  `decision:adopt-di/container`.

## Acceptance

- The hub, the runner host, and the CLI are wired only through their containers.
- The container-only gates fail on a planted bag parameter and on a collaborator constructed outside a provider.
- Behavior is unchanged at every route and command.
