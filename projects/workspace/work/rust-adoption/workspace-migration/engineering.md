# Workspace adoption engineering requirements

For `epic:rust-adoption/workspace-migration`.

## Inspectable migration boundary (engineering:migration-boundary)

Migration rejects unknown schema versions and computes its intended changes without applying them during inspection.
Every input format accepted at cutover has a conversion or compatibility fixture; unknown fields cannot disappear as a
side effect of parsing. Serves `product:adoption-preview`.

## Recoverable conversion (engineering:recoverable-conversion)

File conversion preserves an inspectable pre-change copy or equivalent recoverable content and records completed steps
before reporting partial failure. Injected write, hook, and interruption failures must leave either the old usable
content or a diagnosed recoverable transition, not truncated authoritative configuration. Serves
`product:owned-content-survives`.

## No implicit legacy execution (engineering:no-legacy-execution)

Neither inspection nor conversion invokes the old Python CLI to interpret a workspace, complete a missing capability, or
repair a failure. Serves `product:explicit-cutover` and `decision:rust-replacement`.

## Consumer contract evidence (engineering:consumer-contract-evidence)

Migration acceptance checks each supported consumer against the replacement's command, configuration, path, and output
contracts with representative integration fixtures or executable probes. A rewritten reference alone is not proof that
an editor or automation can parse and use the selected component. Serves `product:consumer-transition`.
