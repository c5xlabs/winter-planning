# Context quality requirements

For `epic:agent-ready-operation/context-quality`.

## Select the content to check (product:quality-content-scope)

An author can check project content by repository, feature environment, several names, or matching names without
accidentally checking a different ownership layer.

Acceptance:

- The default is the current feature environment; outside one, it is all feature environments.
- Explicit project-repository selection checks its source checkout; environment selection checks its project worktrees.
  Workspace-root governance and standalone installed clones are not silently included as ordinary default targets.
- Ambiguous repository/environment names, unknown literal names, and conflicting scope selectors are refused; a glob
  matching no names is a clearly reported successful no-op.
- The result names the selected content and its source; source precedence follows
  `decision:agent-ready-operation/context-quality/check-source-precedence` rather than silently substituting bundled
  help or another checkout.

## Check dirty and unpushed work (product:quality-changed-content)

An author can check the files they are about to publish without scanning every repository.

Acceptance:

- Changed scope includes staged, unstaged, and untracked work plus files changed in commits not yet pushed to the
  upstream, using the configured main remote reference when no upstream is available.
- Names containing spaces or non-ASCII characters remain distinct, and a rename is checked at its current location.
- An empty changed set is a successful no-op that does not invoke external checks with an empty scope.
- Outside a repository, or when the changed set cannot be established reliably, the caller is told rather than shown a
  misleading clean result.

## Understand the check catalog (product:quality-check-catalog)

An author knows which convention checks ran and can distinguish them from workspace health probes.

Acceptance:

- Built-in coverage includes module dependency/reference extractability, context-size limits, and required-service
  references where the relevant declarations exist.
- Dependency checks identify cycles and unavailable or undeclared dependencies, while respecting intentionally
  illustrative examples and code blocks rather than treating them as live dependencies.
- Context-size checking distinguishes eagerly injected guidance from reference material and accounts for content rather
  than formatter padding; configurable limits and both effective and raw sizes are explainable in a finding.
- File-size discovery skips untracked ignored files, retains tracked files, and does not drop candidates when ignore
  lookup is uncertain; other checks do not silently inherit that discovery exclusion.
- No checks contributed, unavailable checks, clean checks, and suppressed findings are distinguishable outcomes.

## Add trusted scoped checks (product:quality-contributed-checks)

Workspace and extension owners can add convention checks over the author's selected content without replacing the
built-in catalog.

Acceptance:

- Contributions identify their owner, accept file and directory scopes, and can query declared module dependencies
  without recursively starting quality checking.
- Missing, non-executable, escaping, unresponsive, failing, or malformed contributions are reported without discarding
  findings from other sources.
- Contribution guidance requires the selected scope to be honored and identifies these as trusted executable programs,
  not filesystem-sandboxed checks or a promise that their code cannot write.
- Structured and human results distinguish check failures from content findings; warnings alone allow a successful run
  and surviving failures make it unsuccessful.

## Suppress only the intended findings (product:quality-scoped-exemptions)

An author can exempt known fixtures or illustrative content while keeping unrelated checks live.

Acceptance:

- Repository-owned exemptions travel with the checked content; workspace-owned exemptions can cover third-party or
  manifest-free repositories and the workspace's own explicitly selected content.
- Rules can select paths for all checks or narrow paths to a named check; relative patterns do not hide findings in
  another repository, and single-segment wildcards do not silently cross directory boundaries.
- Repository and workspace exemptions union; neither can re-enable what the other legitimately exempts.
- Findings without a file, including check-execution failures, cannot be hidden by a path exemption.
- An exemption suppresses a finding after checking; it does not skip execution or authorize a checker to ignore scope.

## Audit exemptions and incomplete runs (product:quality-exemption-accountability)

An author can see what a clean result excluded and repair declarations that no longer tell the truth.

Acceptance:

- Suppressed findings never fail the run, are always counted, and can be inspected together with the matching rule.
- A rule matching no content, or suppressing nothing in content actually checked, warns; a rule whose existing content
  was outside a narrow run does not generate a false stale warning.
- Unknown keys, unusable values, and unreadable ignore declarations identify their owning source and suppress nothing;
  severity follows `decision:agent-ready-operation/context-quality/ignore-failure-severity`.
- Each selected scope has its own result; machine-readable no-op and multi-scope results remain machine-readable.
- Counts name what they measure under `decision:agent-ready-operation/context-quality/contributor-accounting`; missing
  built-in coverage is never reported as checks that ran clean.
