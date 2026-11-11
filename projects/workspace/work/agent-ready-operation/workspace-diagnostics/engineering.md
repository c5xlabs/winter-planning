# Workspace diagnostics engineering requirements

For `epic:agent-ready-operation/workspace-diagnostics`.

## Read authority for built-in probes (engineering:builtin-diagnostic-authority)

Built-in health execution has no implicit reconciliation, write, resource-allocation, or executable-extension discovery
authority. Reading tool availability or native repository state cannot trigger configured setup actions. Serves
`product:diagnostic-repair-separation`.

## Contribution containment and attribution (engineering:health-contribution-execution)

External health entry points are checked for executable eligibility and resolved containment within their declaring
workspace or extension directory, including symlink targets, before launch. Invocation supplies the explicit selected
workspace/extension context through the external-code boundary, not imported code or the legacy CLI. Serves
`product:contributed-health-observations`.

## Isolated finite contribution outcomes (engineering:health-contribution-failures)

Each health contribution has finite execution and output bounds under
`decision:agent-ready-operation/workspace-diagnostics/contribution-limits`; exceeding a bound, launch failure, invalid
output, and unsuccessful exit remain attributable outcomes distinct from reported health observations. Failure in one
contribution cannot erase accepted observations from another. Serves `product:contributed-health-observations` and
`product:actionable-health-outcomes`.

## Preserve evidence availability (engineering:health-prerequisite-accounting)

The diagnostic result retains unavailable prerequisites and unattempted coverage rather than converting them into empty
successful collections; projection checks compare against the same effective declarations and rendered intent used by
their owning capabilities. Startup behavior follows
`decision:agent-ready-operation/workspace-diagnostics/degraded-startup`. Serves `product:workspace-health-inventory` and
`product:diagnostic-startup-failure`.
