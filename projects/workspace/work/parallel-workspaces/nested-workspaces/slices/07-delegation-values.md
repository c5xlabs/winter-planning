---
name: delegation-values
depends-on:
  - module-scaffold
---

# The delegated slice, keys, and footprint

Adds `Slice`, `slice`, `delegated`, `inherited`, and `footprint` of [delegation.md](../spec/delegation.md), Domain and
Values: the pure rules that split the outer env's port band into `K + 1` equal slices, name each child's service prefix,
choose the outer local keys a child inherits, and size a child's shared-plus-feature footprint. Delivers the allocation
rules of `product:nested-runtime-allocation` and the key selection of `product:nested-local-inheritance`, under
`decision:parallel-workspaces/nested-workspaces/delegation-policy`.

## Change

- `crates/core/src/nested/delegate.rs` holds the four functions and `Slice`; `port_base` is the built derivation of
  configuration-model.md, Derived values, passed in as `env_port_base`.
- `inherited` reads only keys of the outer local table passed in, never `project_repository` or `standalone_repository`,
  and copies each value whole.
- No caller yet.

## Acceptance

- Unit tests of `slice`: `ports_per_env` 20 with two nested projects gives slices of 6 at `env_port_base + 6` and
  `+ 12`, the remainder unused; a band smaller than `K + 1` gives slices of size 0.
- Unit tests of `delegated`: `base_port` and `service_prefix` `<service_prefix>-<env>-<project>` always; with `envs` 3,
  also `env_aliases = []` and `envs_per_workspace = 4`.
- Unit tests of `inherited`: unset `inherit_local` copies `git` alone; `[]` copies nothing; a named key absent from the
  outer local table copies nothing for it; `project_repository` named explicitly is not copied; a table is copied whole.
- Unit tests of `footprint`: values from the local layer win over the child's committed file; a boolean or non-integer
  falls back to the next layer, then to 48 and 20.
- `cargo dev gate slice` exits 0.
