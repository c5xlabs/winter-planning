---
name: ref-tokens
depends-on:
  - module-scaffold
---

# Ref token expansion

Adds `ref_tokens::expand` of [ref-tokens.md](../spec/ref-tokens.md): the pure expansion of `{main}`, `{master}`, and
`{default}` against each repository's own main branch. This is the contract `ws merge` calls here and `ws checkout`,
`ws reset` of `epic:parallel-workspaces/safe-history` and `ws restack` of `epic:parallel-workspaces/stacked-history`
call after this epic. Delivers `product:parallel-workspaces/ref-tokens` and its bullet of `product:select-git-targets`.

## Change

- `crates/core/src/ref_tokens.rs` holds `ALIASES` and `expand`: tokens scanned left to right as the reference's
  `(?<!@)\{([^{}]*)\}`, each alias replaced by the main branch, the first unknown token or an empty main branch refused,
  every other character kept, and the result not scanned again.
- No command calls it yet.

## Acceptance

- Unit tests over every row of the spec's table: `{main}`, `origin/{default}`, and `{master}..HEAD` expand; `HEAD@{1}`
  comes back byte for byte; `origin/{mian}` is `Unknown` naming `{mian}`.
- Unit tests: `@{u}` and `main@{yesterday}` hold no token; `{a{main}` expands its `{main}`; `{}` is `Unknown` with an
  empty name; a ref with an alias and an empty main branch is `NoMainBranch`; a ref with two unknown tokens names the
  first.
- `cargo dev gate slice` exits 0.
