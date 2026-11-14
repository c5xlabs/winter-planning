# Executable contribution decisions

## Scaffold command (decision:agent-ready-operation/executable-extensions/scaffold-command)

**Decider:** product owner

**Open.** Choose whether `ext new` is ported or dropped as a registered deviation; the owner leans to dropping it. This
blocks the `ext new` row of the command surface spec and the scaffold's file set, not any other extension command, and
is settled when `epic:agent-ready-operation/executable-extensions` is refined.

## Shared protocol integration (decision:agent-ready-operation/executable-extensions/protocol-integration)

**Decider:** Platform and Workspace engineering with the product owner

Workspace keeps the reference tool's extension contract: the `winter-ext.toml` manifest and the `service` slot at `v1`,
with an absent version treated as compatible. It consumes no shared Platform protocol.

## Execution policy (decision:agent-ready-operation/executable-extensions/execution-policy)

**Decider:** Workspace engineering and the product owner

External programs run as the reference tool runs them: the parent environment and standard input inherited, no output
limit, and no timeout except the 30 s cap on an env band command entry. Standard input is closed only where the
reference tool closes it, for an env band command entry and for an init run started inside the dashboard; the invocation
contract spec holds the per-seam rows.

## Hook order and failure policy (decision:agent-ready-operation/executable-extensions/hook-policy)

**Decider:** product owner and lifecycle engineering

Hooks run in adoption order, and a non-zero exit is a repository error naming the hook and its code. All hooks are
skipped when `adopt_extensions = "none"`.

## Conformance authorization (decision:agent-ready-operation/executable-extensions/conformance-policy)

**Decider:** extension-contract engineering and the product owner

`ext verify` runs the extension's real entrypoint in the live workspace with no isolation, as the reference tool does.
Its checks are those of the slot's spec.
