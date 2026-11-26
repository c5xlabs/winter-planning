# Ref tokens

The expansion of `{main}`, `{master}`, and `{default}` in a ref the engineer types, against each repository's own main
branch. `ws merge` ([merge.md](./merge.md)), `ws checkout` and `ws reset` of `epic:parallel-workspaces/safe-history`,
and `ws restack` of `epic:parallel-workspaces/stacked-history` expand through it; each caller's leaf says which refs it
expands and when. Reference locations are paths under `tools/winter-cli/src/winter_cli/` of `winter-workspace` at
`127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/ref_tokens.rs — product:parallel-workspaces/ref-tokens; pure: no git, no file, no network
pub(crate) const ALIASES: [&str; 3] = ["main", "master", "default"];

pub(crate) fn expand(r#ref: &str, repo: &str, main_branch: &str) -> Result<String, RefTokenError>;

// crates/core/src/termination.rs — CommandError gains RefToken(#[from] RefTokenError), exit 1
#[derive(Debug, thiserror::Error)]
pub enum RefTokenError {
    // "unknown ref token '{<token>}' in <ref!r>; accepted aliases: {main}, {master}, {default}"
    Unknown { token: String, r#ref: String },
    // "repository <repo!r> has no main_branch configured; cannot expand '{<token>}' in <ref!r>"
    NoMainBranch { repo: String, token: String, r#ref: String },
}
```

## Interfaces

```text
# reference: modules/workspace/ref_resolver.py:9-12,42-76
token        "{" then any run of characters holding neither "{" nor "}", then "}", where the "{" does not directly
             follow "@"; the reference's regex (?<!@)\{([^{}]*)\}, scanned left to right, each match ending
             where the next scan starts
             so `@{u}`, `HEAD@{1}`, and `main@{yesterday}` hold no token and reach git as typed; in `{a{main}` the
             token is `{main}`; `{}` is a token whose name is empty
expand       each token, in order of position:
               name in ALIASES                  -> replaced by main_branch, as loaded for the repository
               otherwise                        -> RefTokenError::Unknown, at the first such token
               main_branch empty                -> RefTokenError::NoMainBranch
             every other character is kept as typed; a ref with no token comes back unchanged, byte for byte
             the result is not scanned again
callers      expand every target repository's ref before the first git call of the command, so a refusal leaves
             every repository untouched (decision:parallel-workspaces/partial-execution); a caller with no target
             repository expands nothing and refuses nothing
results      events and documents carry the ref as typed; a caller's per-target record may add that target's
             expansion, as ws reset's repo_reset.ref does (product:parallel-workspaces/ref-tokens)
```

| Typed              | `main_branch` | Expansion        |
| ------------------ | ------------- | ---------------- |
| `{main}`           | `master`      | `master`         |
| `origin/{default}` | `trunk`       | `origin/trunk`   |
| `{master}..HEAD`   | `main`        | `main..HEAD`     |
| `HEAD@{1}`         | any           | `HEAD@{1}`       |
| `origin/{mian}`    | any           | Unknown `{mian}` |
