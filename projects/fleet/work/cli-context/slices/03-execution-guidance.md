---
name: execution-guidance
depends-on:
  - context-command
---

# Publish delivery, worker, and runner-host context

Deliver the execution and delivery branches of `product:task-taxonomy` and the remaining role/landing behavior of
`product:factory-operation` through [Markdown sources](../spec.md#markdown-sources). Apply
`engineering:context-example-proof` and `engineering:context-usage-boundary` to keep guidance separate from runtime
data.

## Change

- Add `delivery.md`, `workers.md`, and `runner-hosts.md`; route them from the root and from other included topics where
  their reader's task needs them.
- Explain delivery evidence and landing outcomes, leased-step usage of Blizzard's worker commands, and the distinction
  between inspecting runner availability and administering the host responsible for execution.
- Teach artifact and prompt interfaces without importing their data into context. Keep restart impact, role authority,
  local/remote targeting, and the CLI-versus-hub version boundary explicit.
- Reuse authoritative Markdown and validate example surfaces according to [packaging](../spec.md#packaging) and
  [command examples](../spec.md#command-examples). Leave existing factory behavior unchanged.

## Acceptance

- The installed CLI reads all three topics offline and follows each advertised route. A build with this guidance but
  without the work branch still has a complete, valid catalog with no dangling work-topic links.
- Coverage fixtures pin policy/check/mergeability diagnosis, external merge versus graph completion, worker assignment
  versus interactive participation, remote availability versus host-local control, and fleet-wide restart impact.
- Example-surface validation accepts the shipped delivery, worker, and host command forms without running mutations or
  calling a real fleet; concrete deployment identifiers, credentials, and runtime record payloads are absent.
- Artifact and prompt commands retain their existing contracts; context topics explain their use but do not resolve
  their names or require worker identity.
