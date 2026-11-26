# Resolution

How the selected bands of one scope become one ordered set of variables: band selection, precedence, references,
visibility, the fixpoint, and its failures. The bands are the `EnvBands` of
[configuration-model.md](../../../../delivered/parallel-workspaces/workspace-configuration/spec/configuration-model.md),
Tables; the base variables are the managed variables of [scope-identity.md](./scope-identity.md); running a command
entry is [value-lookups.md](./value-lookups.md). Reference locations are paths under `tools/winter-cli/src/winter_cli/`
of `winter-workspace` at `127cdf2`. Message wording is the binary's own (`decision:functional-parity`).

## Domain

```rust
// crates/core/src/scope_env/resolve.rs — pure apart from the lookup port; engineering:scope-computation-authority
pub enum Lookups { Run, Mask }      // Mask: no command entry runs (product:offline-env-inspection)
pub const PLACEHOLDER: &str = "<unresolved:command>";

pub struct ScopeEnv { pub vars: Vec<(String, String)> }  // ordered; keys unique

pub fn resolve(
    bands: &EnvBands,
    identity: &Identity,
    lookups: Lookups,
    run: &mut dyn FnMut(&LookupRequest<'_>) -> Result<Vec<(String, String)>, LookupError>,   // value-lookups.md
) -> Result<ScopeEnv, ResolutionError>;

type Position = (u8, usize);               // (band rank, index within its band); base variables rank below all

#[derive(Debug, thiserror::Error)]
pub enum ResolutionError {
    Token { band: String, key: String, token: String },       // the token as declared, never a rendered value
    Offset { band: String, key: String, name: String, offset: String },
    ShellReference { band: String, key: String, name: String },
    Unresolved(Vec<Unresolved>),                              // sorted by key, then by referenced name
    Lookup(LookupError),
}
pub struct Unresolved { pub band: String, pub key: String, pub cause: UnresolvedCause }
pub enum UnresolvedCause {
    Undefined { name: String },             // nothing visible to this entry can ever supply it
    Unclaimed { name: String, why: Unclaimed },
    Cycle { name: String },                 // name reaches itself through pending references
    Blocked { name: String },               // name is pending for a reason its own line gives
    NotVisible { name: String },            // resolved, but not a workspace-band key, read by a workspace-band entry
    ShellWithheld,                          // a shell entry the multi-value bar held back for the whole run
}
pub enum Unclaimed { Anonymous, NotExported, MultiValueLabel(EnvFormat), NotInOutput }
```

Visibility: `resolve`, `Lookups`, `PLACEHOLDER`, and `ScopeEnv` are `pub` inside their private module and re-exported by
name from `lib.rs`, the names of `scope_env/resolve.rs` on a `pub use` line of their own, with every type their
signatures name: `ResolutionError`, `Unresolved`, `UnresolvedCause`, and `Unclaimed`, which the scaffold re-exports
(`context/architecture/changes.md`, A domain module, of `winter-workspace`), so the slice that lands each tests it
before a command calls it (`context/standards/testing.md`) and the gate finds nothing unused without an `allow`
(`context/standards/rust.md`). `resolve` is re-exported as `resolve_scope`, since `pins::resolve::resolve` holds
`resolve` at the crate root. Everything else here stays `pub(crate)`.

## Interfaces

### Band selection and precedence

```text
# reference: modules/workspace/env_provisioner.py:155-182, modules/workspace/env_band_resolver_service.py:1101-1128
scope Workspace   bands: workspace (rank 0, label "env.workspace.vars")
scope Feature(n)  bands: workspace (rank 0), feature (rank 1, "env.feature.vars"), named band n (rank 2, "env.<n>.vars")
                  a named band for another name is never read; a named band for a name nothing registers is inert
effective entries for each band by rank, each entry by index: effective[key] := (label, value, position, restricted)
                  restricted := rank 0. A later band's entry replaces the earlier one for the same key, which is never
                  rendered and never run. A key keeps the place of its first declaration in effective order.
```

### References

```text
# reference: modules/workspace/env_band_resolver_service.py:322-411
a template is a literal entry's string, or a command entry's `command` field
reference   "${" NAME "}" | "${" NAME "+" N "}"     NAME = [A-Za-z_][A-Za-z0-9_]*   N = one or more ASCII digits
token       any other "${" … "}" in the declared template, up to the first "}"
            -> ResolutionError::Token, checked on the declared template when the entry is attempted
            registered deviation, decision:running-environments/scope-environment/persistent-value-exposure: the
            reference scans the rendered text, so a substituted value holding "${" was re-read and printed
render      each reference is replaced by the visible value of NAME, inserted as text; nothing rescans the result
            (product:layered-runtime-values)
${NAME+N}   decision:running-environments/scope-environment/resolution-compatibility
            the value of NAME matches ^[+-]?[0-9]+$ and it and the sum fit i64 -> the sum, decimal, "-" when negative
            otherwise ResolutionError::Offset { name, offset: N }; the value is not carried
            (registered deviation: the reference prints the value in this message)
            NAME gated (Mask, below) -> PLACEHOLDER, whatever N
```

### Visibility

```text
# reference: modules/workspace/env_band_resolver_service.py:664-758
base names    the five managed names, WINTER_PORT_BASE included at both scopes
unrestricted  sees every value accepted so far (resolved)
restricted    sees the base variables as identity() computed them, minus WINTER_PORT_BASE, plus the value its own
              band last claimed for each key whose effective entry is restricted and is not a base name; never a
              feature or named value, whatever later overwrote that key
self-read     an entry that declares a base name sees that base variable's value under its own name
              (WINTER_SERVICE_PREFIX = "${WINTER_SERVICE_PREFIX}-x"); a restricted entry never sees WINTER_PORT_BASE
seeding       resolved := the base variables, except a base name some effective entry declares, which is withheld
              until that entry's claim lands; every base variable holds the lowest position
```

### The fixpoint

```text
# reference: modules/workspace/env_band_resolver_service.py:830-1099
Mask   before the loop, every command entry's key that does not start "_" claims PLACEHOLDER at its position and is
       gated: a reference to it never waits and renders PLACEHOLDER. No command entry runs or is pending. A key
       starting "_" is neither claimed nor gated, so a reference to it fails as Unclaimed { Anonymous } in both modes.
       A key a dotenv or json entry would import is unknown under Mask: a reference to it is Undefined
       (product:offline-env-inspection: never an empty string)
Run    every effective entry is pending

round  pass 1  each pending entry that is not shell = true, in pending order
       pass 2  each shell = true entry, in position order. One with a reference of its own waits for a later round
               while any other dotenv or json entry is still pending; one with no reference is attempted at once
attempt
       a reference neither visible nor gated -> the entry stays pending
       literal  render; claim(key)
       command  shell = true and a reference names a key now held by command output -> ResolutionError::ShellReference
                render, run (value-lookups.md), parse, filter; for each output (name, value):
                  dropped when an effective literal declares name at a higher position
                  else claim(name); an accepted claim marks name command-held
       a literal's accepted claim clears its key's command-held mark
claim(key, value, position)
       accepted when nothing claimed key, or its last claim's position is lower; a rejected claim is dropped
       a restricted entry's claim also updates the restricted view under its own position record
halt   a round that accepts nothing with entries still pending -> ResolutionError::Unresolved, one line per pending
       entry and missing name, classified in this order: not reachable by the entry -> Undefined; an effective
       command entry that already ran -> Unclaimed (why: key starts "_" -> Anonymous; outside its exports ->
       NotExported; dotenv or json -> MultiValueLabel; else NotInOutput); name in a cycle -> Cycle; name pending ->
       Blocked; else NotVisible. A pending entry with no missing name -> ShellWithheld
order  ScopeEnv.vars: the managed variables in scope-identity.md's order, then the band keys in effective order, then
       keys first accepted from command output, in acceptance order. A withheld base name takes its managed place.
       The order is not contract: consumers read the variables as a set
known limitation, kept
       an entry that reads a key before a higher-position command output overwrites a lower literal renders the
       literal's value; the key's own final value is the command's (resolution-compatibility)
```

### Key validation at load

```rust
// crates/core/src/config_model/env_bands.rs — extends EnvVarFault of configuration-model.md; checked first, for a
// literal and a command entry alike. Registered deviation, decision:running-environments/scope-environment/resolution-compatibility
pub enum EnvVarFault {
    InvalidKey,     // the key does not match ^[A-Za-z_][A-Za-z0-9_]*$; ConfigError::EnvVar names band and key
    // … the variants configuration-model.md lists, unchanged
}
```
