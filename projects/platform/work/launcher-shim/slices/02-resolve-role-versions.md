---
name: resolve-role-versions
depends-on:
  - production-process-contract
---

# Resolve exact versions from pins, ancestors, and user defaults

Implement `resolve` and version validation from [Domain](../spec.md#domain), [Flows](../spec.md#flows), and the
selection records in [Data](../spec.md#data). Deliver `product:respect-component-version` and the selection cases of
`engineering:fixture-only-dispatch` under `decision:launcher-shim/version-scope` and
`decision:launcher-shim/version-selection`.

## Change

- Validate exact release strings before any cache-path join and retain the winning source file in `Selection`.
- For Workspace only, discover the nearest `.winter/config.toml`; a valid pin skips ancestor selections, while an absent
  pin falls through without consulting an outer workspace config.
- Merge every existing ancestor `.winterrc` from filesystem root through cwd independently per role. Validate even
  values later shadowed by children.
- Consult the requested role's user default only when no applicable pin or directory value exists.
- Return `NoSelection` with searched paths or `InvalidConfig` with file and cause; do not infer versions from PATH or
  cache contents. Keep this resolver unconnected to public command dispatch.

## Acceptance

- Isolated root/intermediate/cwd fixtures prove per-role inheritance, child overrides, absent keys, and winning-source
  provenance. HOME affects discovery only when its directory is an actual ancestor.
- Workspace pin fixtures prove precedence, nearest-config stopping, and fallback on an absent pin; Hub and Runner never
  discover or read even malformed workspace configuration.
- Missing user files/keys produce `NoSelection`; independent defaults work outside a workspace and lose to directory
  selections. Defaults are not read when a higher-precedence selection exists.
- Malformed TOML, invalid recognized versions, unreadable files, and lookup failures other than not-found produce
  `InvalidConfig`, including shadowed ancestor values. Unknown tables and keys remain ignored.
- Grammar tests cover release/prerelease forms and reject ranges, aliases, leading-zero numeric parts, and path-shaped
  values before path construction.
- The complete discovery tree is injected or isolated, including ancestors above the fixture directory; tests neither
  consult real ancestor selections nor modify configuration.

## Out of scope

- Cache validation and missing-release errors belong to `prepare-cached-dispatch`; XDG capture belongs to
  `wire-launcher-dispatch`.
