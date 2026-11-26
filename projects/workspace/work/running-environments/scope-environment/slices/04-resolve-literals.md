---
name: resolve-literals
depends-on:
  - scope-identity
---

# Resolving literal entries, and masking command entries

Adds `resolve`, `Lookups`, `PLACEHOLDER`, and `ScopeEnv` of [resolution.md](../spec/resolution.md) for every entry but a
command entry that runs: Band selection and precedence, References, Visibility, and The fixpoint's `Mask` setup, pass 1
over literals, `claim`, the halt classification, and the order. Under `Mask` no command entry runs, so this resolves
every scope `winter env` prints without `--resolve`. Delivers the layering bullets of `product:layered-runtime-values`,
the masking bullets of `product:offline-env-inspection`, and `engineering:scope-computation-authority`, under
`decision:running-environments/scope-environment/resolution-compatibility` and
`decision:running-environments/scope-environment/persistent-value-exposure`, whose token check on the declared template
and offset error without the value are registered deviations.

## Change

- `scope_env/resolve.rs` holds `resolve` over `EnvBands` and an `Identity`: the bands by rank, the effective entries,
  the restricted view of rank 0, seeding with withheld base names, the rounds, and `ScopeEnv` in the spec's order.
- The template grammar: `${NAME}` and `${NAME+N}`, any other `${…}` a `Token` error checked on the declared template;
  rendering inserts values as text and never rescans; an offset follows the decimal rule of resolution-compatibility.
- Under `Mask`, each command entry's key not starting `_` claims `PLACEHOLDER` and is gated; a key a dotenv or json
  entry would import is `Undefined`.
- The render function is `pub(crate)`, so `run-lookup` renders a command's words with it.
- Under `Run`, a command entry is attempted by `resolve-command-entries`; until it lands, no caller passes `Run`.
- `resolve`, `Lookups`, `PLACEHOLDER`, and `ScopeEnv` are re-exported by name from `lib.rs` for its tests
  ([resolution.md](../spec/resolution.md), Domain); no command calls them yet.

## Acceptance

- Unit tests of band selection: the workspace scope reads the workspace band alone; `Feature(alpha)` reads workspace,
  feature, and `env.alpha.vars`, never `env.beta.vars`; a later band's entry replaces the earlier and keeps the first
  declaration's place.
- Unit tests of references: a reference to a key declared later resolves; `${PORT+1}` over `"4020"` is `"4021"` and over
  `"-1"` is `"0"`; over `"1_0"`, `" 1"`, and a sum past `i64` it is `Offset` carrying no value; `${a-b}` and `${}` are
  `Token` with the token as declared; a substituted value holding `${X}` is inserted as written.
- Unit tests of visibility: a workspace-band entry reading a feature key is `NotVisible`, and one reading
  `WINTER_PORT_BASE` never sees it; `WINTER_SERVICE_PREFIX = "${WINTER_SERVICE_PREFIX}-x"` renders the managed prefix
  plus `-x`; a plain value redeclaring `WINTER_ENV_INDEX` replaces it in place.
- Unit tests of the halt: an undefined name, a two-entry cycle, and an entry blocked by the cycle each answer their
  `UnresolvedCause`, sorted by key then name, and no `ScopeEnv` is returned.
- Unit tests under `Mask`: a command entry's key holds `PLACEHOLDER`, and `${KEY+1}` renders `PLACEHOLDER`; a reference
  to a `_` key is `Unclaimed { Anonymous }`; a reference to a key only a dotenv entry would import is `Undefined`, never
  an empty string; no `run` callback is called.
- `cargo dev gate slice` exits 0.
