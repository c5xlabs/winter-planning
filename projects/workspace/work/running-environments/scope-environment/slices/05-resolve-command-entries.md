---
name: resolve-command-entries
depends-on:
  - resolve-literals
---

# Resolving command entries under `Run`

Adds the command half of [resolution.md](../spec/resolution.md), The fixpoint, under `Lookups::Run`: every effective
entry pending, a command entry attempted through the `run` callback with its `LookupRequest`, its filtered output
claimed or dropped against higher literals, the command-held marks, pass 2 of `shell = true` entries with the
multi-value bar and `ShellReference`, and the `Unclaimed` and `ShellWithheld` lines of the halt. The callback is a test
double here; `compute-print` binds it to `run_lookup`. Delivers the claim and refusal bullets of
`product:external-value-resolution`, the blocked-reference bullet of `product:layered-runtime-values`, and the
provenance of `engineering:value-provenance`, under
`decision:running-environments/scope-environment/resolution-compatibility`, whose intermediate read is kept as the known
limitation.

## Change

- `resolve` under `Run` attempts each pending command entry once its references are visible, in pending order, with
  `visible` its entry's view; a `LookupError` ends the run as `ResolutionError::Lookup`.
- An output name is dropped when an effective literal declares it at a higher position, else claimed; an accepted claim
  marks the name command-held and a literal's accepted claim clears it.
- Pass 2: a `shell = true` entry with a reference of its own waits while any other dotenv or json entry is pending; a
  reference to a command-held key is `ShellReference` before the entry runs.
- Keys first accepted from command output follow the band keys in acceptance order.

## Acceptance

- Unit tests with a recording callback: a raw entry's value is claimed under its key; a dotenv entry's two names are
  claimed and readable by a later literal; an entry referencing an imported name runs after the import.
- Unit tests: an output name a higher literal declares is dropped and the literal's value stands; an output name a lower
  literal declares is replaced.
- Unit tests of the halt over entries that ran: a reference to a raw entry's own key that its `exports` leaves out is
  `Unclaimed { NotExported }`, to a dotenv or json entry's own key `Unclaimed { MultiValueLabel }` with its format, and
  to a raw entry's key its filtered output did not hold `Unclaimed { NotInOutput }`.
- Unit tests of shell entries: one referencing a dotenv import waits for it; one referencing a command-held key is
  `ShellReference`, and the callback is never called for it; a shell entry held back by a pending import that never
  resolves is `ShellWithheld`.
- A unit test: a callback answering a `LookupError` ends `resolve` with that error and no `ScopeEnv`; later entries are
  not attempted.
- A unit test of the known limitation: an entry that read a lower literal before a higher command output replaced it
  renders the literal's value, and the key's final value is the command's.
- `cargo dev gate slice` exits 0.
