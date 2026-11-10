# Platform decisions

## Launcher language (decision:launcher-language)

The platform executable and its shim are Rust from day one, rather than starting with a Bash launcher, so the first
implementation establishes the language and executable boundary intended for the released tool. The code lives in
`c5xlabs/winter`, separate from the Workspace and Fleet implementations.

## Shared version cache (decision:version-cache)

Released component binaries use the shared version cache at `~/.cache/winter/versions/` by default, following the XDG
cache location rather than creating `~/.winter/versions/`. An explicit `XDG_CACHE_HOME` relocates it to
`$XDG_CACHE_HOME/winter/versions/`; mutable source-build output is not a released-version cache entry.
