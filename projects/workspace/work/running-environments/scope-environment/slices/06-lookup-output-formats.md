---
name: lookup-output-formats
depends-on:
  - module-scaffold
---

# Reading a lookup's output

Adds `read_output` of [value-lookups.md](../spec/value-lookups.md), Domain, the parse and filter of its Formats: a
finished lookup's stdout read as `raw`, `dotenv`, or `json` into named values, then filtered by the `_` and `WINTER_`
prefixes and the entry's `exports`. Pure: it reads bytes and starts nothing. Delivers the format and allow-list bullets
of `product:external-value-resolution` and the parse-error bullet of `product:private-runtime-values`, under
`decision:running-environments/scope-environment/resolution-compatibility`, which keeps the reference's dotenv parsing
and not its Python JSON spellings.

## Change

- `scope_env/lookup.rs` gains `read_output`, the parse of each format and the filter, failing as `NotUtf8` or
  `Output(OutputFault)`; no fault carries an output line or value.
- A JSON number keeps the text the program wrote; `crates/core/Cargo.toml` gains what `serde_json` needs for that.
- `read_output` is re-exported by name from `lib.rs` for its tests (value-lookups.md, Domain); no command calls it yet.

## Acceptance

- Unit tests, raw: stdout with surrounding blanks and a trailing newline answers the key with the trimmed text.
- Unit tests, dotenv: blank and `#` lines are skipped; `A = 1`, `A="x y"`, and `A='x'` answer `1`, `x y`, and `x`;
  `A="x'` keeps its quotes; `A=1 # c` answers `1 # c`; a `\r\n` file reads as `\n`; a repeated name takes the last value
  at the first place; `FOO`, `export FOO=1`, and `=1` are `DotenvLine` with their 1-based line and no text.
- Unit tests, json: `{"A":"x","B":true,"C":null,"D":1e16,"E":0.10}` answers `x`, `true`, `null`, `1e16`, and `0.10`; an
  array is `JsonNotObject { "array" }`; `{"a-b":1}` is `JsonKey`; `{"A":{}}` is `JsonNotScalar`; `NaN` and a truncated
  object are `JsonSyntax` with a position-only cause.
- A unit test: non-UTF-8 stdout is `NotUtf8` in each format.
- Unit tests of the filter: `_X` and `WINTER_X` are dropped in every format, a raw entry's key included; with `exports`
  declared, a name outside it is dropped; output order is kept.
- `cargo dev gate slice` exits 0.
