# Service orchestration decisions

## Built-in and external provider coexistence (decision:running-environments/service-orchestration/builtin-external-coexistence)

**Decider:** product owner

**Open.** Choose how a built-in tmux, docker, or process-compose provider coexists with an external extension that
provides the same backend: naming, precedence, and whether a built-in binds without a declaration. This blocks
finalizing `engineering:service-provider-conformance` and provider dispatch in the three provider epics, not the
external provider contract; it is settled when `epic:running-environments/service-orchestration` is refined.

## Readiness enforcement (decision:running-environments/service-orchestration/readiness-enforcement)

**Decider:** product owner and engineering

Readiness follows the reference tool: `service up --wait` polls status and passes as soon as no service reports
`unhealthy`, so an unknown, empty, or unparseable status passes. No provision state is recorded, and nothing gates
`service up` on an earlier `provision`.

## Log selection policy (decision:running-environments/service-orchestration/log-selection-policy)

**Decider:** product owner and engineering

Logs follow the reference tool: without `--follow`, providers are read one after another and one tail limit applies to
the combined lines; with `--follow`, lines are merged in arrival order with no backlog re-ordering. Time filters apply
only to lines that carry a timestamp.

## Service manifest quoting (decision:running-environments/service-orchestration/service-manifest-quoting)

**Decider:** engineering

The service manifest file quotes its strings with the reference tool's string representation, not TOML quoting. Existing
providers parse that form, so the binary writes it byte for byte.

## Built-in execution placement (decision:running-environments/service-orchestration/builtin-execution-placement)

**Decider:** product owner

The tmux, docker, and process-compose providers are built into the binary and need no Python. The external provider
contract is kept unchanged, and external extension code still runs out of process.

## Provider activation (decision:running-environments/service-orchestration/provider-activation)

**Decider:** product owner and engineering

Binding follows the reference tool: an explicit `[capabilities] service` list is validated in order, and without one a
sole candidate binds implicitly and several all bind, sorted by name. An undeclared spec version is compatible, and the
`--service-orchestrator` option wins over `WINTER_SERVICE_ORCHESTRATOR`, which wins over configuration.

## Declaration continuity (decision:running-environments/service-orchestration/declaration-continuity)

**Decider:** product owner and engineering

Contributed service definitions reach a provider only on `service up`, through the manifest file. Their availability to
later actions is the provider's own state, as in the reference tool.
