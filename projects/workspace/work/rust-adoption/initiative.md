---
initiative: rust-adoption
refinement: outline
epics:
  - workspace-migration
  - parity-validation
  - workspace-release
---

# Adopt the Rust workspace

Existing users need an explicit, evidenced transition from the Python tool to an independently installable Rust
component.

## The problem

An engineer cannot judge a replacement by its language or command count. Their repositories, local settings, provider
behavior, agent tooling, and maintained consumers must remain usable, and installing a new executable must not become an
unreviewed conversion of a live workspace. Demonstrating successful setup alone does not establish the refusal and
preservation guarantees the engineer relies on.

## The outcome

`epic:rust-adoption/workspace-migration` previews the transition, preserves user-owned content and feature work, and
coordinates changes to supported integrations. `epic:rust-adoption/parity-validation` accounts for the source-derived
capabilities and deliberate differences with isolated behavioral evidence. `epic:rust-adoption/workspace-release`
integrates a complete prebuilt Workspace component with shared suite release enablement and explains upgrade cost
without requiring Fleet or a compiler.

These epics consume the outcomes of `initiative:parallel-workspaces`, `initiative:running-environments`, and
`initiative:agent-ready-operation`. Together the four initiatives reach `milestone:rust-workspace-parity`; rough parity
is the shared destination, not a separate initiative or proof supplied by publication alone.

## Stance

Local evaluation and disposable previews can proceed before public delivery. Live replacement cutover and a release
claiming rough parity require accepted capability, provider, migration, and handoff evidence with no unresolved blocking
redesign. The Rust component never forwards unfinished commands to Python; the old CLI may remain independently selected
until deliberate cutover. Release governance and the common CLI runtime baseline are inherited from the suite, not
redefined here.
