# Winter component install engineering requirements

For `epic:component-install`.

## What already holds

- `decision:release-automation/release-automation` publishes Rust CLI archives and checksums as GitHub Release assets
  through `cargo-dist`, on the four targets of `decision:release-automation/release-targets`.
- `decision:launcher-shim/component-discovery` runs a component only from `<role>/<version>/<target>/` in the version
  cache, after its `component.toml` validates. A directory without a valid manifest is never run.
- The launcher shim's spec resolves a role's selection from the nearest `.winterrc` and the user `config.toml` under
  `decision:launcher-shim/version-scope`. Installation reads selections through that resolution and writes neither file.

## Brief requirements

- engineering:shared-selection — `winter install` with no component resolves each role through the launcher's own
  selection code, not a second implementation. Serves `product:install-selected-versions`.
- engineering:cached-fetch-offline — Fetching a release already in the cache makes no network request. Serves
  `product:install-component-version`.

## Verified installation (engineering:verified-install)

A release becomes a cache entry only after its archive's SHA-256 matches the published checksum, under
`decision:component-install/release-source`, and the entry appears by one atomic rename of a fully extracted directory.
Concurrent fetches of the same missing release download it once: one holds a per-entry lock, the others wait for it and
then report the entry as present. Serves `product:verified-install`.

Acceptance:

- A checksum mismatch, truncated download, or failed extraction exits `1` and leaves no entry and no temporary files in
  the cache.
- Interrupting a fetch with `SIGINT` or `SIGTERM` mid-download leaves no entry; the next fetch starts the download
  again.
- Eight concurrent fetches of a missing release against a counting fixture release source produce one archive download
  and one entry.
- A lock left by a killed process does not block a later fetch beyond the lock-wait timeout.
- An existing entry is never rewritten or deleted by a fetch of another release or component.

## Download bounds (engineering:download-bounds)

**Proposed.** A release download fails if connecting takes over 10 s or no bytes arrive for 30 s, and a waiter gives up
on another fetch's lock after 300 s. Each failure names the release, the source URL, and the bound reached. Serves
`product:explain-install-failure`.

## Controlled install verification (engineering:fixture-release-source)

Install verification uses a fixture release source and fixture cache, configuration, and workspace directories; it never
contacts GitHub, and never reads or modifies the user's real version cache, `.winterrc`, or user `config.toml`. Serves
`product:install-component-version`, `product:install-selected-versions`, and `product:verified-install`.

Acceptance:

- The fixture release source serves archives and checksums for several components, versions, and targets, can serve a
  corrupted or truncated archive, and counts requests.
- The fixture release source can answer an unknown version and a version with no archive for the host target, and the
  two produce different errors.
