---
name: classify-native-routes
depends-on:
  - production-process-contract
---

# Classify native routing tokens without parsing application tails

Implement the classifier specified in [Interfaces](../spec.md#interfaces), serving `product:legacy-workspace-dispatch`,
`product:legacy-init`, `product:launcher-verbosity`, `product:reach-component`, and `engineering:fixture-only-dispatch`.
Follow `decision:launcher-shim/routing-boundary`; keep the classifier unconnected to the executable until all adapters
exist.

## Change

- Recognize only an optional first `--winter=PATH`, preserve native path bytes, and consume no subsequent token for a
  malformed control.
- Skip exact leading `--verbose`/`-v` for classification while retaining them in legacy argv.
- Recognize modern groups, init, sample, platform controls, and registered platform commands only at the leading
  platform position; select a route once and leave its tail opaque.
- After an opaque Python option prefix begins, scan only for an exact legacy command name without an option schema,
  option-arity tracking, or value decoding.
- Preserve foundation parsing for inspection-only combinations and usage errors; reject incompatible source overrides
  before any lookup and retain the init index needed for the later token rewrite.

## Acceptance

- Table-driven tests cover every example in the spec's invocation table and every allowed legacy command.
- Modern arguments preserve spaces, empty/non-UTF-8 values, option-like tokens, delimiters, and application
  help/version; root verbosity is not included in their tails.
- Legacy args preserve the entire original vector except the consumed initial source control, including root verbosity,
  unknown options, equals/separate values, nested commands, and non-first `--winter` tokens.
- Python option values containing init, role names, platform controls, or sample never activate those routes. Scanning
  stops at a selected legacy command; later tokens cannot reclassify it.
- Empty/missing-equals source controls and incompatible modern overrides fail with usage exit 2 without source lookup.
  Override-only invocation is an error; bare/verbosity-only invocation selects successful help.
- Foundation help/version aliases and standard combined/control-flag behavior remain supported; production rejects
  sample while development retains its own adapter and raw tail.

## Out of scope

- `wire-launcher-dispatch` activates the classifier, advertises the complete implemented surface, and injects real
  environment/process adapters.
