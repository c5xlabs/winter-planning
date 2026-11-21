---
name: reference-scenarios
depends-on:
  - freeze
  - fingerprint-command
  - epic:rust-adoption/parity-validation
---

# Recorded scenarios of the input commands

Records `ws update`, `ws update --freeze`, and `ws fingerprint` against the reference tool and replays them against the
binary, proving the commands' flows in [standalone-pins.md](../spec/standalone-pins.md) and
[content-identity.md](../spec/content-identity.md) end to end under `decision:functional-parity`.

## Change

- Scenarios under `scenarios/ws-update/` and `scenarios/ws-fingerprint/`, recorded with `cargo dev record` from the
  reference tool, each with a fixture workspace whose standalones clone a fixture remote; a remote tag is moved by raw
  git calls. No scenario declares a `[[stub]]`.
- No production code changes.

## Acceptance

- `ws-update` scenarios: a declaration changed to a newer tag; a tag moved on the remote, reported up to date; a branch
  pin with a local-only commit reset to `origin/<ref>`; a second run up to date; a dirty refusal; an unknown literal
  name.
- `ws-update` freeze scenarios: a pin written into `config.toml`, one into `config.local.toml`, a dirty refusal, and
  `--force`. Each declares `[[compare.file]] path = "ws/.winter/*.toml" mode = "bytes"` with the reason that the
  `Set a standalone ref` writer keeps the file's other bytes, which the parsed-TOML default would not show.
- `ws-update` drift scenarios: a declared project missing on disk writes the drift notice before the run, with and
  without `--freeze`.
- `ws-fingerprint` scenarios: a clean workspace and one with a staged edit, each giving the reference's digest.
- `cargo dev gate full` exits 0.
