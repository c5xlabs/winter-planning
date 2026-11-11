# Dependency graph requirements

For `epic:agent-ready-operation/dependency-graph`.

## Inspect declared dependencies (product:declared-module-dependencies)

An author can inspect extension-eligible modules and their declared dependencies without reconstructing the inventory.

Acceptance:

- Installed extension modules and project repositories carrying an eligible module declaration can appear as nodes;
  ordinary repositories are not invented as modules merely because they are present.
- Each module's outgoing dependencies reflect its declaration, including references to unavailable targets.
- A module with no declared dependencies remains visible; an empty inventory is distinguishable from failed discovery.
- Inspection does not install modules, execute extension code, repair declarations, or change repositories.

## Reuse the dependency picture (product:dependency-query-consumption)

People and checks can consume the declared module graph through a non-interactive query.

Acceptance:

- A readable listing and machine-readable consumption are available without opening the dashboard.
- Repeated queries against unchanged inputs present a deterministic module order and retain declared dependency order.
- Checks can obtain graph data without invoking content checking again or requiring a separately installed legacy CLI.

## Identify incomplete evidence (product:dependency-evidence-completeness)

An author knows when a missing or unreadable declaration prevents the query from describing all eligible modules. The
handling choice is owned by `decision:agent-ready-operation/dependency-graph/incomplete-inputs`.

Acceptance:

- An unreadable eligible declaration identifies its module and cause rather than appearing as an empty dependency list.
- Consumers can distinguish a complete graph from partial or unavailable evidence.
- Reporting available nodes does not imply that missing modules have no dependencies or that the workspace passed lint.
