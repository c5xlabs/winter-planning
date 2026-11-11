# Context quality engineering requirements

For `epic:agent-ready-operation/context-quality`.

## Scope evidence cannot fail silently (engineering:quality-scope-evidence)

Changed-set discovery preserves native path identity, deduplicates current paths, and treats a failed status/diff or
unavailable comparison base as unavailable evidence rather than an empty changed set. Conflicting selectors are rejected
before contribution execution. Serves `product:quality-content-scope` and `product:quality-changed-content`.

## Check-source provenance (engineering:quality-source-provenance)

Every run identifies the checked content revision/location separately from the source of its rules and executable
contributions. Version-bound framework guidance remains governed by context navigation; inspecting working project
content must not silently replace it with bundled documentation or a source checkout. Serves
`product:quality-content-scope` and `product:quality-scoped-exemptions`.

## Built-in availability is independent (engineering:quality-catalog-availability)

A missing or failed built-in check cannot disable unrelated built-in checks or turn unavailable coverage into zero clean
contributors. Built-ins execute as first-party Rust behavior under the project runtime constraint, without a separately
installed graph executable or bundled Python checker. Serves `product:quality-check-catalog`.

## Trusted contribution execution bounds (engineering:quality-program-boundary)

Contributed entry points resolve within their declaring directory, including symlink targets, and run through the
external-code boundary with explicit selected scope and source attribution. Execution and output have finite bounds;
launch, output, timeout, and exit failures remain unsuppressible non-file failures, while valid findings survive a later
program failure. Serves `product:quality-contributed-checks`.

## Fail-safe exemption matching (engineering:quality-exemption-safety)

Exemption matching is relative to the owning checked repository or explicitly declared workspace surface, uses complete
segment-aware matches, and resolves unusable rules to no suppression. Rule auditing cannot recurse indefinitely through
symlinked directories; partial scopes are accounted for before judging a live rule stale. Serves
`product:quality-scoped-exemptions` and `product:quality-exemption-accountability`.
