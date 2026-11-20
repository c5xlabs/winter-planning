# Replacement validation decisions

## The recorder is Rust and shares the replay code (decision:rust-adoption/parity-validation/recorder-placement)

**Decider:** engineering

The record step is a subcommand of the repository's development binary and uses the same fixture builder, snapshot,
normalization, and comparison code as replay, treating the reference tool as an opaque executable whose path the caller
supplies. A recorder in Python or shell was rejected: it would put a second implementation of the fixture and the
snapshot on a branch that carries no Python.

## Scenario state is built, never produced by a tool (decision:rust-adoption/parity-validation/scenario-state-prerequisites)

**Decider:** engineering

A scenario's starting state comes only from its fixture description; a scenario that needs the outcome of an earlier
command lists that command as a compared step of the same scenario. Preparing state by running the reference tool at
record time and the binary at replay time was rejected, because a fault in the preparing command would then surface as a
failure of the command under test.

## TOML files compare as TOML, other files as bytes (decision:rust-adoption/parity-validation/state-comparison)

**Decider:** engineering

A file a scenario changes or creates is compared as parsed TOML when its name ends in `.toml` and byte for byte
otherwise, and git state as normalized text, because `decision:toml-parity` makes a TOML file's values its function and
other files are user data. A scenario compares a TOML path by bytes only where a made decision fixes its lines, naming
that decision as the rule's reason, and may relax any path to presence or to ignored, with the reason beside the rule.

## The two proof behaviours (decision:rust-adoption/parity-validation/proof-behaviours)

**Decider:** engineering

The method is proven on `ws index` and `repo remove`, each under `--json`, as the method proof spec lists them: the
first reads two configuration keys and the state registry with three distinct outcomes, the second edits one committed
file in place and has failure paths. Every smaller candidate either changes no state or already needs the whole
configuration model, the lock file, or a remote.

## Program capture is built with the first non-git program (decision:rust-adoption/parity-validation/program-capture-placement)

**Decider:** product owner

Stub programs, their call records, and the `Calls` comparison are specified in this epic, in the `[[stub]]` table and
Program calls of [scenario-format.md](./spec/scenario-format.md) and the `Calls` row and `stub.rs` of
[record-and-replay.md](./spec/record-and-replay.md), and are built by
`epic:agent-ready-operation/executable-extensions`, the first epic whose commands start a program other than git. Until
then the harness refuses a scenario with a `[[stub]]`. No command built before that epic starts such a program, and the
git calls of earlier commands are verified by their effect on fixture repositories, not by their command lines.

## Parity evidence is qualified at release (decision:rust-adoption/parity-validation/evidence-ownership)

**Decider:** product owner

This epic delivers the scenario method and its proof, which every command epic uses to record its own behaviour against
the reference tool. The account of the whole replacement's coverage, its trial on representative disposable work, and
its failure and concurrency evidence need commands no epic has built yet, so they belong to
`epic:rust-adoption/workspace-release`, where a rough-parity release claim is qualified:
`product:rust-adoption/workspace-release/capability-accounting`,
`product:rust-adoption/workspace-release/isolated-adoption-proof`, and
`engineering:rust-adoption/workspace-release/adversarial-scenarios`. `engineering:disposable-runtime-proof` stays here:
the test harness of `epic:parallel-workspaces/component-runtime` and the scenario format are built on it, and later
runtime scenarios apply it.
