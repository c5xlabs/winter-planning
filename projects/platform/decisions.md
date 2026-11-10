# Platform decisions

## Separate development from release delivery (decision:release-delivery-separation)

**Decider:** product owner and engineering

`epic:cli-foundation` delivers the local Rust development and verification foundation, while `epic:release-automation`
owns publishing, packaging, signing, and release-policy gates so rapid development does not wait for distribution
readiness. Neither the foundation nor `epic:launcher-shim` depends on that release epic; unreleased development builds
are not published-release compatibility baselines.

## Launcher language (decision:launcher-language)

The platform executable and its shim are Rust from day one, rather than starting with a Bash launcher, so the first
implementation establishes the language and executable boundary intended for the released tool. The code lives in
`c5xlabs/winter`, separate from the Workspace and Fleet implementations.

## Shared version cache (decision:version-cache)

Released component binaries use the shared version cache at `~/.cache/winter/versions/` by default, following the XDG
cache location rather than creating `~/.winter/versions/`. An explicit `XDG_CACHE_HOME` relocates it to
`$XDG_CACHE_HOME/winter/versions/`; mutable source-build output is not a released-version cache entry.
