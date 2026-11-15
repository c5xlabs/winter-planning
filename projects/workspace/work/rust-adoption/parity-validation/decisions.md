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

## State is compared by bytes unless a scenario says otherwise (decision:rust-adoption/parity-validation/state-comparison)

**Decider:** engineering

Files a scenario changes or creates are compared byte for byte, and git state as normalized text, because committed
configuration files are edited in place and their comments and layout are user data. A scenario relaxes one path to
parsed TOML, to presence, or to ignored, with the reason beside the rule, for bytes that `decision:functional-parity`
does not class as function.

## The two proof behaviours (decision:rust-adoption/parity-validation/proof-behaviours)

**Decider:** engineering

The method is proven on `ws index` and `repo remove`, each under `--json`, as the method proof spec lists them: the
first reads two configuration keys and the state registry with three distinct outcomes, the second edits one committed
file in place and has failure paths. Every smaller candidate either changes no state or already needs the whole
configuration model, the lock file, or a remote.
